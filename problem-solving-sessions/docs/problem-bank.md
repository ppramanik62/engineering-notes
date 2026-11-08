# The problem bank

A pool to draw from, not a course to work through. There's no order, and nothing here has to get used.

If you turn up with a problem of your own, we do that instead. Something you ran into and wondered about is worth more than anything on a list. I'll also make problems up when a session leaves something worth coming back to, so plenty of days the problem won't be from this file at all.

The levels are a rough sense of how much is going on, not a ladder. The focus tag says what kind of thinking a problem leans on, and gives nothing away, so read this file whenever you like.

| Level | What it's like |
|---|---|
| 1 | Small and self-contained. Most of the work is seeing what to keep track of |
| 2 | More than one sensible way to do it |
| 3 | The constraints pull against each other, so something has to give |
| 4 | Open-ended. Part of the job is deciding what the problem even is |

## Level 1

| # | Problem | Focus |
|---|---|---|
| 1 | One elevator in a ten-floor building. Someone presses a button inside, someone presses one on a floor | state |
| 2 | Tic-tac-toe: tell me if the last move won the game | representation |
| 3 | A deck of cards: shuffle it, deal seven, put a card back at the bottom | representation |
| 4 | An undo button for a drawing app | state |
| 5 | A cinema seating chart that takes bookings and shows what's free | representation |
| 6 | A calendar with repeating events, like "every second Tuesday until June" | state |
| 7 | A leaderboard that shows the top ten and tells each player their own rank | representation |
| 8 | A traffic light at a four-way junction, with a button for pedestrians | state |
| 9 | Snake and ladder, but the board is three levels stacked on top of each other | representation |

## Level 2

| # | Problem | Focus |
|---|---|---|
| 10 | A text editor that opens a 2GB file and lets you type in the middle of it | representation |
| 11 | A cache that holds 1000 things and throws out the one you used least recently | budget |
| 12 | Autocomplete for a search box: 10 million possible terms, 50ms to answer | budget |
| 13 | A spreadsheet that recalculates the right cells when one cell changes | representation |
| 14 | A chess board that can list every legal move | representation |
| 15 | A file browser that answers "every file over 1GB under this folder" | representation |
| 16 | A rate limiter: 100 requests a minute per user, spread across 20 servers | time |
| 17 | A URL shortener, worked out from what it has to store rather than from the usual answer | state |
| 18 | A train network that answers "cheapest route from here to there" | representation |
| 19 | An ad server picking which ad to show, from 5 million, in 10ms | budget |

## Level 3

| # | Problem | Focus |
|---|---|---|
| 20 | A job queue with retries, where a job must never run twice | failure |
| 21 | A payment API where the client retries and must not be charged twice | failure |
| 22 | Notes syncing between two phones that were both edited offline | time |
| 23 | Ticket booking for a show that's about to sell out | time |
| 24 | A chat app's unread count, correct across three devices | time |
| 25 | A lock held across machines that has to expire if the holder dies | failure |
| 26 | A metrics system taking a million points a second, queried over a year | budget |
| 27 | Deduplicating ten billion URLs with 8GB of memory | budget |
| 28 | A feature flag service where some staleness is fine, and you decide where | time |
| 29 | A multiplayer game where two players are 100ms apart and both shoot first | time |
| 30 | A database that has to survive the plug being pulled mid-write | failure |

## Level 4

| # | Problem | Focus |
|---|---|---|
| 31 | Index a codebase so anyone can search it by name and by meaning | open-ended |
| 32 | A knowledge graph over git commits and AI coding sessions. What are the nodes, really | open-ended |
| 33 | Give an AI agent the right context when you only have so many tokens to spend | open-ended |
| 34 | Regenerate a per-folder `AGENTS.md` every night, incrementally, from a diff | open-ended |
| 35 | Spot near-duplicate AI coding sessions. You decide what makes two of them the same | open-ended |
| 36 | A hosted code index for many customers: one index each, or one index with a customer column | open-ended |
| 37 | A build system that never rebuilds anything it doesn't have to | open-ended |
| 38 | Answer "who broke this test, and when" over a year of commits | open-ended |
| 39 | A graph database on object storage, where every read costs 50ms | open-ended |
| 40 | Something you're working on that you haven't designed yet. You pick | open-ended |

## Adding to this

Anything real counts. Tell me and I'll add it, or just bring it to a session and we'll start there.
