---
title: "FalconUp 2026 - what-if portfolio plan"
date: "2026-07-17"
description: "Milestone 7 kickoff: the risk engine only ever sees the portfolio you already own — planning the hypothetical portfolio, and the five edge cases that decide the API shape"
dayNumber: 21
milestone: 7
milestoneTitle: "what-if portfolio"
---

milestone 6 ends with a portfolio risk engine that tells you how risky your
portfolio is and gives it a grade

which is useful exactly once. you look at the C, you read the breakdown, and
then the obvious next question is the one the app cant answer: *ok, so what do
i do about it*

### what milestone 7 adds

a what-if portfolio. you edit your holdings hypothetically — what if i bought
more of this, what if i sold that — and the app re-evaluates the milestone 6
risk engine against the edited version instead of the real one

the point is that the grade stops being a verdict and starts being a dial. C
with "too concentrated in semis" underneath it is a diagnosis. C that becomes
B when you sell a third of SOXL is a decision

and the nice part is that milestone 6 already did the hard half. the risk math
is pure functions over holdings — no database, no network, holdings in results
out. it does not care whether the holdings it got are the ones in the table. so
milestone 7 is not new math at all. its a new way to build the input

### what a what-if actually is

same holdings list, with a delta applied per symbol. three outcomes:

- symbol you already own → apply the delta to its shares
- symbol you dont own → new row
- symbol you sold entirely → row disappears

thats the whole model. its a transform from holdings to holdings, which means
the hypothetical portfolio and the real portfolio are the same shape, which
means the risk engine takes either one without knowing the difference

nothing gets written. the real holdings table is untouched

### the cases that decide the API

writing the happy path first is how i end up with an endpoint that returns a
confident risk grade for a portfolio that cant exist. so, the five cases i want
pinned down before any of it gets written:

<u>buy a symbol you dont yet have</u>

→ new symbol buy. valid, just creates a row instead of editing one

<u>buy a symbol thats not in the database</u>

→ 404, `symbol not tracked`

only 16 symbols got seeded back in milestone 3, and the risk math needs price
history for every holding it sees. a symbol with no history isnt a small
problem, its a correlation matrix with a hole in it. so this gets rejected at
the edge rather than being allowed in and degrading something three layers down

<u>sell more than you own</u>

→ 400, with a message saying so

no negative share counts. theres no short position concept anywhere in this
app, and silently clamping to zero would be worse than rejecting — the user
asked for something impossible and should be told, not quietly given a
different portfolio than the one they described

<u>sell everything</u>

→ the symbol disappears from the portfolio entirely

not a row with 0 shares. zero-share rows would leak into every weight
calculation in milestone 6 as a denominator term that contributes nothing

<u>sell down to exactly one symbol</u>

→ correlation matrix shows "unavailable" instead of crashing

this is the interesting one. correlation needs at least two things to correlate.
one holding isnt an error the user made, its a portfolio thats legitimately
allowed to exist — it just has no pairwise structure to measure

and this is already covered. milestone 6's tests pinned the degenerate cases
specifically because of this — one holding, two identical holdings, no price
history. what changes in milestone 7 is that the degenerate case stops being a
test fixture and becomes something a user can produce in two clicks

so the rule is that the rest of the risk report still renders. concentration
still works with one holding, leverage still works, beta still works.
correlation returns unavailable and the grade is computed without it. one
missing metric shouldnt take the page down

### order of operations

same as the last two milestones — backend first, frontend last. the delta
transform and its validation before anything that renders

next: building it
