# Questions worth asking

Not a checklist. These are the questions that tend to get you unstuck, roughly in the order they're useful.

## What does this actually need to remember?

Read the problem and write down only what it mentions. If a fact can be worked out from other facts, you probably don't need to store it. If you are storing it anyway, there's usually a reason, and it's worth saying what it is.

The thing to watch for: nouns that come from how something is drawn or displayed rather than from the problem. A board printed as a 10 by 10 grid isn't necessarily a 2D problem.

## What do you have to do with it, and how often?

List the operations. Then mark which ones happen constantly and which happen once. The frequent one usually decides the shape of everything else.

## What's the simplest thing that could work?

An integer. An array. A map. Start there on purpose. Then ask what breaks it. Quite often nothing does, and you're done.

A good sign you've picked the wrong representation: you're writing conversion code to make it fit.

## What does this cost?

Rough numbers beat adjectives. Not "it's fast" but "100 cells at one byte each, so the whole thing is 100 bytes, and a move is one add and one lookup".

Memory, time per operation, and the amount of fiddly code you have to write to keep it correct. That last one counts.

## What would break this?

Ten times the data. A thousand times. The input arriving in a weird order. A machine dying halfway. The rare operation suddenly becoming the common one.

Sometimes the answer is "nothing I care about", and that's a real answer.

## Is there another way to look at it?

Not every problem needs a second design. But when the first one feels like hard work, it's usually worth asking what a completely different representation would do to the problem. Often the work disappears instead of getting easier.
