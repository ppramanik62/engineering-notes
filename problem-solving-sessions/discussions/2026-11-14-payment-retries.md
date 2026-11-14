# 2026-11-14: a payment that is retried

## The problem

A shop's app calls our API: "charge this customer £20 for order 7731". We charge the card and reply "done". On a bad connection the reply is sometimes lost. The app sees a timeout and sends the same request again. Nobody may be charged twice for one order. What does the server have to remember to tell a retry from a new payment, and what happens when the server itself dies halfway?

## How you thought about it

You started with the right idea in one line: the client makes a unique key for each payment and sends the same key on every retry.

Then you wrote the whole flow down. On the first request: authenticate, then in one transaction insert `(customer_id, idempotency_key, request_fingerprint, status)` under a unique constraint on `(customer_id, idempotency_key)`, commit with `processing`, call the provider with the same key, update the status from the provider's answer, reply. On a retry: the insert hits the constraint, the server loads the row and compares fingerprints, and returns the saved result, or "processing", or rejects a different request.

I asked about the crash: the server commits `processing`, calls the provider, and dies. The row says `processing` for ever. You added a background worker that looks for rows stuck in `processing`, asks the provider about each, copies a final answer, leaves a still-running one alone, and retries the provider call with the same key when the provider has no record.

Then I changed the provider. No idempotency keys, only "charge" and "list charges". You said the worker can no longer charge again on "not in the list", and that a human has to look. You also saw that the list can fail. I added that a charge in flight may not be listed yet, so "absent" is ambiguous. You answered with the fix: attach our own reference to the charge so the list can be searched by it, and wait the provider's documented maximum delay before believing an absence. You ended with "everything depends on the provider", which is true.

## What worked

The unique constraint as the guard. You did not check and then insert. You inserted and let the database refuse the second one. That is why the retry that arrives while the first request is still running cannot also charge, and it needed no lock.

The fingerprint. A key reused for a different payment is a real client bug, and you caught it in the first sketch without being asked.

Not holding the transaction open across the provider call. You committed `processing` first, so the row exists before anything external happens. That is what lets the worker find the request later.

On the provider without keys, you refused to guess. You named the cost difference without prompting: a delayed payment is cheap, a double charge is not.

## Where it got stuck

The first design had no answer for a worker that dies after `processing` is committed. The retry finds `processing` and nothing will ever change it. A small question, "how do you tell still working from died?", got you to reconciling against the provider.

The last bullet of your first sketch, "if the server itself timed out, retry the provider with the same key", was the start of the same idea. It covered a timeout inside a live request but not a dead server. The worker is that bullet, made to run on its own.

You did not retell it in your own words at the end. You asked for the note instead. The retelling is still to do.

## The design we landed on

**What's stored.** One table, and the client supplies the key.

```text
payment                 unique (customer_id, idempotency_key)
  id, customer_id, idempotency_key,
  request_fingerprint,  hash of amount, currency, order, card
  status,               processing | succeeded | failed | needs_review
  provider_ref,         our own reference sent with the charge
  result,               the saved response, once final
  created_at, updated_at
```

`provider_ref` is our own id for the payment (the row id works). It is sent as the provider's idempotency key when the provider has one, and as charge metadata always.

**The first request.**

1. Authenticate the customer.
2. In one transaction, insert the row with `status = processing`. Commit. No network call happens inside the transaction.
3. Call the provider with `provider_ref`.
4. Set `status` and `result` from the answer. Reply.

**A retry with the same key.** The insert fails on the constraint, so the server loads the row.

| Row says | Same fingerprint | Different fingerprint |
|---|---|---|
| `succeeded` or `failed` | return the saved result | reject, the key belongs to another request |
| `processing` | return "processing", or wait briefly | reject |
| `needs_review` | return "pending review" | reject |

**The worker.** Every minute, for rows in `processing` longer than a set time:

```text
ask the provider about provider_ref
  charged      -> succeeded, save the result
  failed       -> failed, save the result
  in progress  -> leave it, look again later
  no record    -> provider has keys:    call charge again with the same key
                  provider has no keys:  wait the documented maximum delay,
                                         ask again, then needs_review
```

With provider keys, "no record" is safe to act on even if the worker is wrong and the original is only slow. The provider sees one key twice and charges once. So "too long" can be a rough number.

**A provider with no keys.** Not finding a charge does not prove none was made. The charge may be in flight, or the list may have failed. The worker waits the provider's documented maximum delay, queries the list by `provider_ref`, and only then marks `needs_review`. A person decides, with the customer's bank statement as the last check. This costs a human. The alternative is a possible double charge.

**What it costs.**

- A row per payment attempt, kept as long as a client might retry, and longer for audit.
- A payment can sit in `processing` for a minute or more after a crash. The client sees "processing", not a wrong answer.
- The result is only as exact as the provider. With keys it is exact. Without them it falls back on our own reference in the metadata, the provider's delay and a person.
- The worker is a second writer to the row. Its update must only move a row out of `processing`, so it cannot overwrite a result the live request just saved.

## Worth coming back to

The row's `processing` state is a stored "I don't know yet". Compare it with the cinema lock, which was a mechanism that died with the process: here the "I don't know" lives in a durable row, which is why a worker can find it after the server dies. Tell me in your own words why that difference matters. Then try a problem that looks different but needs the same thing: a job queue where a job must never run twice.
