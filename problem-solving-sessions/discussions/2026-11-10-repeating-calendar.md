# 2026-11-10: a calendar with repeating events

## The problem

A calendar for one person. Events can happen once ("dentist, 14 March, 10:00"), every weekday ("standup at 09:30"), or every second Tuesday until the end of June ("book club"). A week view and a month view show any week, past or future. Two edits people always want: move just one occurrence ("this Tuesday's book club to Wednesday"), and change a series from a date on ("standup at 09:45 from next Monday").

## How you thought about it

You started from the data again: days, each holding an array of events, each event a title, a time and a duration with a one-hour default. You picked structured input over parsing text, which kept the problem on what matters.

Then two things came up at once. You reached for transactions so that a move can't leave zero copies or two copies of an event. And you raised a doubt: "every weekday" can't be written out to the end of time, so you capped it at a year.

That doubt was the whole problem. Your design stored about 260 rows for one standup. When I asked how you'd change all of them from next Monday on, you said you'd loop over them. Then a one-off "Team standup" on a Saturday showed the loop had no way to tell which rows belonged to the series. You added an event_type_id, a name for the thing the person actually created.

For the week 13 months out, you proposed a job that keeps adding occurrences as days pass. When I asked how that job knows what to add, you put the rule into an event_type record. One more question, "what do the 260 rows give you that the rule doesn't?", and you said "nothing really, we can generate occurrences from the rule." The cap and the top-up job went away with the rows.

The rest came quickly. Moving one occurrence became an exceptions map, original date to new date and time. Changing the series from Monday on became a list of rules, each with the dates it covers. When a rule change met an old exception, you said the exception wins, and the design already did that.

## What worked

Saying the doubt out loud. The cap at a year felt wrong to you, and you said so instead of building around it. That was the thread that led to the design.

Keying exceptions by the original date. An occurrence has no id of its own, so "the one that would have been on 14 April" is its name, and you picked that without being pointed at it.

## Where it got stuck

The first design stored results: every occurrence, written out. It took four questions to get from 260 rows to one rule. Each step was yours (the id, the top-up job, the event_type record), but each one kept the rows and added something next to them. What shifted it was asking what the rows held that the rule didn't.

It's the same pull as the undo node holding the whole text after an edit. At the end you named it yourself: both store the change, not the result it produces.

## The design we landed on

**What's stored.** One record per event, one-off or repeating.

```text
event_type
  id
  title
  duration          default 1 hour
  rules             list of { repeat pattern, time, from date, until date or none }
  exceptions        map: original date -> moved to (date, time) | cancelled
```

A one-off event is a rule that happens once. "Every weekday" with no end date is fine, because nothing is written per day.

**Opening a week.** For each event_type, ask each rule which of its dates fall in the seven days. Then apply the exceptions: drop the cancelled ones, move the moved ones (a move can bring an occurrence into this week from another). The cost is the same small amount of work for next week or for a week ten years out.

**Moving one occurrence.** Add one entry to exceptions, keyed by the date it would have had.

**Changing a series from a date on.** End the current rule the day before and add a new rule from that date. Past weeks still show the old time. Google Calendar does the same with "this and following events", except it splits into two series instead of keeping a list of rules.

**When a rule and an exception disagree,** the exception wins. Exceptions sit on the event_type, not on one rule inside it, so swapping rules doesn't touch them.

**What it costs.** You can't list "every event on 3 March" by reading one place. You have to ask every rule. That's fine for one person's calendar. For a thousand calendars shown in one view, it might not be.

## Worth coming back to

The transactions you reached for in your first answer. For one person editing one calendar, a move is a single write to one record, so there's nothing to keep atomic. Worth noticing when a problem needs that machinery and when it only looks like it does, which is the mechanism-versus-data question from the cinema session coming from another side.
