---
title: "FalconUp 2026 - risk math"
date: "2026-07-13"
description: "Milestone 6: building the risk engine bottom-up — result models first, then the math, then tests, then the router"
dayNumber: 17
milestone: 6
milestoneTitle: "portfolio risk"
---

ok so i have seven metrics i want. time to build

same order of operations as milestone 5 — backend first, frontend last. but
this time theres an extra layer before even the backend endpoints, because the
math is the actual thing here and the endpoints are just a wrapper around it

### step 1: create models

define what each result should have, *before* writing the function that
produces it

this sounds backwards but its the step that saved me the most time. if i write
the math first i end up returning a loose dict and then the frontend has to
guess whats in it. writing the shape first means the function has a target

e.g.

```python
@dataclass(frozen=True, slots=True)
class ConcentrationResult:
    """Herfindahl index + sector exposure breakdown."""

    herfindahl_index: float  # 0-1, higher = more concentrated
    top_holding_pct: float  # largest single holding as % of portfolio
    sector_breakdown: dict[str, float]  # sector -> % of portfolio value
    signal: str  # "highly_concentrated", "moderate", "diversified"
```

`frozen=True` so a result cant be mutated after its computed — a risk number
that something downstream can quietly edit is worse than no risk number.
`slots=True` because these get built per request and theres no reason to carry a
dict per instance

notice the result carries both the raw value *and* the `signal`. thats the
thing from the last post — the number is the math, the signal is the
interpretation, and keeping them in the same object means the frontend never has
to re-derive my thresholds

create one for each metric:

- concentration result
- correlation result
- effective leverage result
- portfolio beta result
- max drawdown result
- stress scenario result
- risk grade result

### step 2: risk math

`backend/risk/math.py`

one file, pure functions, no database and no network. holdings in, results out

- hardcoded historical time periods of stress scenarios
- concentration calculation math
- correlation matrix math
- effective leverage math
- portfolio beta math
- max drawdown
- historical stress test
- worst period
- risk grade calculation

the stress periods being *hardcoded* is on purpose. theyre dates, they already
happened, they are never going to change. no reason for them to live in a table

### step 3: tests for math

this is the one milestone where tests were obviously worth it, and its because
of the pure-functions thing above — no db, no mocking, no fixtures. just call
the function with a made-up portfolio and assert the number

and risk math is *silently* wrong in a way most code isnt. if a route breaks i
get a 500. if a correlation matrix is subtly wrong i get a plausible looking
number on a dashboard and i believe it 🫠

the ones worth pinning down are the degenerate cases — one holding, two
perfectly identical holdings, zero price history. those are where a formula
divides by zero or quietly returns something that looks fine

### step 4: risk router

`backend/routers/risk.py`

then register it in `routers/main.py`. thin by design — pull the user's
holdings, pull price history, hand it to the math, return the results

### step 5: frontend

### step 6: update docs

the structure signal exists now
