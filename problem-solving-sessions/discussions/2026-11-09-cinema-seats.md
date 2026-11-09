# 2026-11-09: booking seats for a cinema showing

## The problem

The booking page for one screen, one showing, 200 seats. Someone sees which seats are free, taps the two they want, pays, and gets a ticket with those seat numbers on it.

## How you thought about it

You went to the data first, which is what we talked about yesterday. Fixed array of 200, index plus one is the seat number, each element a struct with availability and price. You said the row layout is a presentation concern and set it aside, which was the right call. You also checked whether tracking revenue was in scope instead of assuming.

When I gave you two people tapping seat 57 at once, you reached for locks straight away: an exclusive lock for the one paying, a shared lock for everyone reading, and a timeout so a lock can't be held forever. The timeout was a good instinct, nobody told you about it.

Then the long part. You had the behaviour right well before the data could express it. You described a seat that reads as booked during the hold and flips back to free when the timeout passes, while the struct still only had a boolean in it. One question about where the expiry time lives and you added the field. From there you listed the three real states cleanly: free, held with an expiry, sold.

The restart question landed well. You said the state has to live in persistent storage and be synced, which is the move that makes the rest of the design work.

The part that took the longest was separating the lock from the hold. You kept the lock spanning the whole five minutes of payment. The timeline with Alice and Bob reading before either wrote got you to the right size of the window, microseconds, and you said so. Then you slid back to the five-minute lock in your next answer. I had to lay the two jobs out side by side before it separated, and then the test settled it in one line from you: "the lock dies with the process, the stored row survives."

The last two answers came fast and were both right: one transaction over both seats, and always take seats in ascending order. The ordering is the same rule you read about with latches in the buffer manager, and you pulled it across without being pointed at it.

## What worked

Going to the data before the mechanism. Yesterday that was the gap, today it was the first thing you did.

Checking scope instead of assuming it. Asking whether revenue tracking was wanted is a small thing that saves real work.

And carrying the fixed-order rule over from the buffer-manager reading. That's the useful kind of transfer, a rule learned in one place used in a place that looks nothing like it.

## Where it got stuck

One thing, and it's worth naming precisely. A lock and a hold are different kinds of thing, and they stayed stuck together for most of the session.

A lock is a mechanism. It lives in memory, it covers a tiny window, and it dies with the process.

A hold is data. It's a row that says "seat 57 is taken until 14:32", it does its job by being read, and it survives a restart.

You had both, but you kept asking the lock to do the hold's job. Two hints didn't shift it. What shifted it was putting the two designs through the same failure: the server dies at minute two.

Worth keeping as a general question: is this thing doing its job by existing, or by running? Those two want very different designs.

## The design we landed on

For future reference, the whole thing in one place.

**What a seat holds.** One row per seat, in persistent storage, not in the process's memory.

```text
seat_number     1 to 200
price
status          free | held | sold
held_until      a timestamp, only set while status is held
held_by         who holds it, only set while status is held
```

**Reading the seat map.** A plain read, no locks. 200 people looking at the page are just 200 reads.

- `sold` shows as unavailable.
- `held` with `held_until` in the future shows as unavailable.
- `held` with `held_until` already past is treated as free.

**Taking seats.** The user taps 57 and 58 and goes to pay.

- One transaction covering both rows, so it's both seats or neither.
- Touch the rows in ascending seat order, so two people asking for the same pair in opposite orders can't deadlock.
- For each seat, write conditionally: set `status = held`, `held_by = this person`, `held_until = now + 5 minutes`, but only if the row currently says free, or says held with an expiry already past.
- If either seat fails that condition, the whole transaction aborts and the user is told. No half booking.

The conditional write is what makes read-then-write one indivisible step. That's the only thing that needed protecting, and it's over in microseconds.

**Payment succeeds.** One transaction: for each seat set `status = sold` and clear `held_until` and `held_by`, only if the row still says held by this person with an expiry that hasn't passed.

**Payment fails, or the user closes the tab.** Nothing to do. The hold expires by itself, because expiry is a timestamp in a row rather than a timer in a process. Readers already treat a past expiry as free, and the next person's conditional write overwrites the dead hold. A background sweep to tidy old rows is optional, not load-bearing.

**Why there's no five-minute lock.** The five minutes of exclusion is done by `held_until` being written down. A lock held that long would be a liability: it dies with the process, and the seat would be stuck or lost depending on which way you guessed.

## Worth coming back to

Two things.

The mechanism-versus-data distinction, since that's where the session stalled. Worth meeting again in a problem that looks nothing like a cinema.

And one hole we never touched: the payment provider confirms six minutes later, after the hold expired and somebody else bought the seat. Money taken, no seat. That's a whole design on its own and a good one for later.
