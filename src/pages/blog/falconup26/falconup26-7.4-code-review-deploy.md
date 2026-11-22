---
title: "FalconUp 2026 - what-if code review"
date: "2026-07-20"
description: "Milestone 7: a code review finds the what-if comparing before and after over two different time windows, which quietly fakes the drawdown improvement — then the deploy"
dayNumber: 24
milestone: 7
milestoneTitle: "what-if portfolio"
---

panel works, numbers move, grade changes. so before merging, a read back
through what i actually wrote

### finding #1 — the window moves

`portfolio_values` gets built in `routers/risk.py`, and the way it handles
missing price data is to only keep dates that *every* holding has data for. the
intersection. which is the right instinct — you cant compute a portfolio value
for a day where you dont know what one of the holdings was worth

the problem is what that does to a what-if

a what-if addition brings in a symbol with its own history length. if that
history is shorter than the rest of the portfolio's, the intersection shrinks,
and the "after" portfolio gets evaluated over a shorter period of time than
the "before" one

so the two halves of the comparison arent measured over the same window. which
breaks exactly one metric, but it breaks it badly

<u>max drawdown</u>

the worst peak-to-trough drop the portfolio would have suffered over a given
time period

the key property is that its monotonic in the window: a bigger time window can
only ever find an equal or worse drawdown, never a better one. more days means
more chances to contain a crash

so shortening the window makes the number go up — toward zero, which the ui
reads as **better**. for free. with no trade having done anything

portfolio risk is graded off five things:

- concentration
- leverage
- correlation
- beta
- **max drawdown**

and four of those are fine here. concentration, leverage and beta are computed
from current weights, not from a price series, so the window doesnt touch them.
correlation already has its own guard. drawdown is the one that reads a time
series end to end, so its the one that silently absorbs the window change

which means the bug doesnt announce itself. nothing errors, nothing renders
blank. you get a confident improvement on a metric that didnt improve

and thats the part i dont love, because last post i pointed at max drawdown
-46.8% → -28.7% as one of the reasons the VOO trade looked good. i dont know
how much of that was real. thats the whole problem with this class of bug — it
doesnt produce a wrong answer you can spot, it produces a plausible one

<u>fix</u>

measure "before" and "after" drawdown over the same window

dont flatten the window just because one holding doesnt have the data. if the
two sides genuinely cant be compared over a common period, say so —
`unavailable` / not comparable

which is a direction the diff entry already has, from the correlation case.
same principle as milestone 7's plan: one metric that cant be computed honestly
returns unavailable and the rest of the report still renders. a missing number
is recoverable, a confidently wrong one isnt

also some dead data in there while i was looking. cleaned up

### merge PR to main

and then the deploy, which is the milestone 6 routine again

<u>update db</u>

```jsx
cd backend
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/falconup uv run alembic upgrade head
```

no new migrations this milestone either — what-if writes nothing by design, so
theres no state to add. still run it, same reasoning as last time: a no-op when
head is current, and running it unconditionally means i never have to remember
which milestones had one

<u>pull containers from ghcr</u>

```jsx
cd ops
docker compose --env-file ../.env down
docker compose --env-file ../.env pull
docker compose --env-file ../.env up -d
```

pulling prebuilt images instead of building on the box, same as every deploy
since milestone 2

### milestone 7 done, properly this time

the feature was finished two posts ago. it just wasnt correct until now, and
the difference between those two states was one read-through
