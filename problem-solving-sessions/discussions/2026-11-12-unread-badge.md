# 2026-11-12: the unread badge

## The problem

A chat app. Messages live on a server. One person uses a phone, a laptop and a tablet, and each conversation shows a badge with its unread count. Reading on one device must drop the badge on the others. A device can be offline for a day and has to be right when it comes back. Two devices can be reading at once. One case to hold: the phone sat in a drawer showing 40 unread, the laptop read 12 of them, and 5 new ones arrived. What does the phone show, and what does it need to get there?

## How you thought about it

You got the number first: 40 - 12 + 5 = 33. (Messages, not conversations. One conversation with 33 unread messages.) Then you said the server is the source of truth and every client is a view of it, which is a good starting shape but only says who is right, not how the phone gets from 40 to 33.

Your first sketch of the server held a list of connected clients with their IP addresses, a list of unread chats, and an array of unread messages inside each. When I asked which part of "how many unread?" needs an IP address, none did, and the clients list disappeared from your next sketch without further prompting.

Your second sketch was data first: a conversation holds messages, a message has sender, receiver, content, timestamp, `sent` and `seen`. Then the laptop's flow: update local state, send a write with 12 message ids, the server validates and sets `seen = true` in one batch. The phone fetches the server state when it opens. Because `seen` sat on the message and belonged to the person, three devices shared one answer.

Then the cost. The phone asking for "everything where `seen = false`" works, but the list screen only needs a number per conversation, and finding that number by filtering means reading every message in 300 conversations each time the app opens. Listing the events that change an unread count (a message arrives, a message is read) got you to the counter: a number per conversation, +1 on arrival, -1 on read.

Two tests of the counter. A retry of the same 12 ids: you first said "it needs to be a transaction", which makes the flag and the counter commit together but does nothing when the first attempt already committed and only the reply was lost. Asking how many rows the second attempt changed got you to the rule: the counter drops by the rows that actually flipped, not by the ids in the request. Then two devices marking message 31 at once: "only one of them can flip it, so only one decrements". Same rule, same answer.

Last, a group of five. You said `seen` has to be per person, then `seen` per (message, person) and a counter per (person, conversation).

## What worked

Dropping the clients array unprompted, and writing the data before the flow. You also kept `seen` on the person, not the device, which is why the phone needs no memory of what it missed.

Seeing that the list screen needs a number, not the messages. That one observation moved the whole cost from the phone to the server and led to the counter.

Putting the rule in terms of what changed. "Only one can flip it" is a sentence about the data. It says nothing about locks or retries, and it covers both.

## Where it got stuck

Twice you gave the same answer to a different question. "Filter `seen = false`" came back after I asked how to avoid reading 8,000 rows, and "a transaction" came back after I asked what a retry does to the counter. Both answers were right for a nearby problem. What moved things each time was a smaller question: list the events that change the count, then how many rows did the second attempt change.

You asked me to write the summary instead of saying it back, so the final retelling in your own words is still to do. It's the first thing to try next time this idea comes up.

## The design we landed on

**What's stored.** The server is the only place that holds read state. Clients keep a copy to draw from and nothing else.

```text
message
  id, conversation_id, sender, content, timestamp

seen                  one row per (message, person)
  message_id, person_id, seen: bool

unread                one row per (person, conversation)
  person_id, conversation_id, count
```

There is no list of devices. A device is a client that asks, not data the answer depends on.

**A message arrives.** One transaction: insert the message, insert a `seen = false` row for each member except the sender, add 1 to each of those members' counters.

**A person marks messages read, from any device.** The client sends the message ids. One transaction: set `seen = true` for those (message, person) rows where it is still `false`. The number of rows that changed, n, is what the counter drops by. Never the number of ids sent.

```text
retry of the same 12 ids      0 rows change     counter drops by 0
laptop and tablet both mark 31   one update flips it, the other changes 0 rows
Bob's counter when Alice reads   untouched, it is a different row
```

**The phone comes out of the drawer.** It asks for the counter of each conversation, about 300 small numbers, and draws them. The server says 33 and that is the whole answer. The phone never compares lists or replays what it missed. When a conversation is opened, it asks for that conversation's messages with their `seen` flags.

**What it costs.**

- Two places say how many are unread, the counter and the flags, and they must agree. The transaction keeps them together on every write. A job that recounts the flags and compares is the way to catch a bug.
- The `seen` table grows by messages times members: 40,000 rows for a five-person group with 8,000 messages, 4 million for 500 people.
- Opening a conversation still reads its messages. Only the list screen avoids it.

## Worth coming back to

The counter is a stored result of the flags, the same kind of thing as the 260 stored occurrences in the calendar, and this time keeping it was the right call. What made it right here and wrong there? One answer to try: the calendar's rows could be derived cheaply from a small rule, and the counter can't be derived without reading thousands of rows. Also unanswered: what `sent` is for.
