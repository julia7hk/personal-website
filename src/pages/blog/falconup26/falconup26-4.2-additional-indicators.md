---
title: "FalconUp 2026 - additional indicators"
date: "2026-07-15"
description: "Milestone 4, revisited: Sortino and max drawdown take the engine from 7 indicators to 9, and the weights get rebalanced so nothing gets double-counted"
dayNumber: 19
milestone: 4
milestoneTitle: "symbol indicator calculations"
---

going back to milestone 4 to add two more indicators to the engine: sortino
ratio and max drawdown

the reason is leveraged ETFs. TQQQ and SOXL are both in the seeded 16, and the
original 7 indicators treat a 3x fund the same as they treat QQQ — volatility
goes in as one number and comes out as one number, with no sense of whether that
volatility was upside or a 70% hole you still havent climbed out of

### the two new ones

<u>sortino:</u> downside volatility only

it's the sharpe ratio with the denominator changed. sharpe punishes *all*
variance, so a fund that rips upward gets dinged for ripping. sortino only
counts the moves against you, which is the kind of volatility anyone actually
cares about

<u>max drawdown:</u> worst peak-to-trough loss

the single number that says "if you had bought at the worst possible moment, how
far underwater did you go." for a leveraged ETF this is the number

### rebalancing the weights

weights have to sum to 1.0, so adding two indicators means taking weight away
from the seven that were already there

- rsi 0.13
- macd 0.13
- bollinger 0.08
- sma 0.13
- atr 0.08
- beta 0.13
- sharpe 0.12
- sortino 0.12
- max_drawdown 0.08

the rationale:

- sharpe drops 0.20 → 0.12 and hands that weight to sortino. they are both
  risk-adjusted-return measures and i dont want to double-count the same idea
  twice at full strength
- drawdown gets 0.08, same as the other volatility measures (bollinger, atr)

### actual building

new models:

```jsx
class SortinoResult:

class MaxDrawdownResult:
```

new math functions in `backend/indicators/math.py`

then redo the composite scoring in `backend/indicators/composite.py` to go from
7 → 9 indicators, and extend the api in `backend/routers/indicators.py`

tests for:

- new endpoints
- modified composite tests
- new indicators

frontend:

- add to the main landing page for each symbol
- composite card
- `frontend/src/types.ts`

run the typescript compiler as a type checker, since none of this is caught at
runtime until a page actually renders

```jsx
npx tsc --noEmit
```

update docs

### address changes from PR review

daddy approved, but with two flags

🟡 the bearish nudge is coming from three places: drawdown, atr, and beta

thats real, but its fine — all three leaning bearish makes the composite more
conservative, and a tool that tells you to slow down on leveraged ETFs is doing
its job

🟡 sortino and sharpe are lowk the same measurement

this one is a fair hit. recalculate them into one vote before retallying, so the
composite isnt counting a single idea twice under two names
