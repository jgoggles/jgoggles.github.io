---
title: Inconsistency Begets Progress
---

**_Discussed:_** _stagnant systems, replacing XYX, predictability, gradual
change, git hygiene, bricks_

<hr />

> A foolish consistency is the hobgoblin of little minds. - Ralph Waldo Emerson

[Creative destruction](https://en.wikipedia.org/wiki/Creative_destruction) is a
process whereby we incrementally replace/render obsolete old innovations with
new ones. This can't happen without a willingness to introduce inconsistency.

Institutional memory is a great driver of excuses. Mainly excuses of why not to
do something. Not doing things is the equivalent of being stagnant. Stagnant
systems are candidates for decay. As makers of software one of our main
responsibilities is to decrease the cost of change. We build things not just for
now but for the inevitable change to come. That change needs to be easy. This
doesn't always happen. Tradeoffs get made, bad code gets written, that code has
momentum, it replicates, becomes part of the codebase's memory, part of the
engineering org's institutional memory. It's ok, it happens everyday. But this
memory, these established patterns, system components resting on complex houses
of cards, these things are hard to change.

One of the main reasons given for not undergoing any type of change in a
codebase, even for the better, is the perceived need for wholesale changes.
Wholesale changes are expensive, they require a lot of planning, cooperation,
digressive action. The thinking is that we can't replace XYZ because XYZ is
everywhere and it would be a whole shitty deal to do that, there might be bugs,
all the usual costs and risks of change. This all-or-nothing point of view comes
from a want for consistency. It's better to have something that is entirely
less-than-ideal than to have more than one way of doing something even if one of
those ways is great.

It makes sense that we strive for consistency as designers of software. We rely
on consistency to create contracts for how code communicates. It would be chaos
otherwise; software wouldn't exist. Consistency means predictability and
predictability is a cornerstone of how abstractions interact. So it makes sense
that software writers operate with consistency as an embedded, innate point of
view. An inconsistent system is not a system at all. But this need for
consistency at the macro level often holds us back when making changes within
the system.

We should be open to gradual change at the cost of consistency for the benefit
of an improved codebase. The model is pretty simple:

1. Any new code should use the new pattern
2. If it's easy, refactor any instances of the legacy pattern as they're touched

This can be implemented as an experiment, meaning a team tries it for a period
of time and reports back on its experiences. If it's positive rinse/repeat the
process org-wide. If it's negative then only a small slice of code needs to be
unwound (source control can help here if the commits are thoughtful and atomic).

Embracing inconsistency as step zero, our codebases becomes better over time. We
establish a north star and leverage our tendency as programmers to reuse what's
there; we just need to lay a single brick to be copied.
