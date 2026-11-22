---
title: "FalconUp 2026 - what-if API layer"
date: "2026-07-18"
description: "Milestone 7: splitting the risk router into fetch, build and compute so the what-if endpoint can reuse the math without touching the database, and pydantic as the gate on anything that takes input"
dayNumber: 22
milestone: 7
milestoneTitle: "what-if portfolio"
---

backend first, like the plan said. and the first thing to build isnt the new
endpoint at all, its a refactor of the old one

### why the refactor comes first

`get_portfolio_risk` in `routers/risk.py` does everything in one function. it
loads holdings from the database, fetches market data, runs the risk math,
returns the report. which is fine when theres exactly one caller

what-if needs three of those four steps, twice, with different holdings each
time. so the compute half has to come out of the request handler before it can
be called from anywhere else

so: extract the compute-from-data logic into a helper

```python
_compute_risk_metrics(data) -> dict
```

concentration, leverage, beta renorm, correlation guard, drawdown, grade. all
of it. then the existing risk router gets edited to just call the helper
instead of doing the work inline

the helper takes an optional extra symbol, for the what-if case

### three functions, not one

splitting it up properly ended up as three pieces inside `risk.py`:

<u>step 1 — `_fetch_market_data`</u>

the part that talks to the outside world

<u>step 2 — `_build_risk_data`</u>

turn holdings plus market data into the shape the math wants

<u>wrapper — `_fetch_portfolio_risk_data`</u>

just a two-in-one shortcut: load portfolio data from the db (step 1), then run
the risk numbers on it (step 2)

and the reason thats three functions instead of one is the call pattern:

- fetching portfolio risk calls the **wrapper** — step 1, step 2
- fetching what-if risk calls **step 1, step 2, step 2**

one fetch, two builds. the market data gets loaded once and then used twice,
against two different holdings lists. the "before" state is computed from the
current holdings only, the "after" recomputes from the modified holdings, and
they share the same price history so the comparison is actually
apples-to-apples

if the wrapper were the only entry point, what-if would be hitting the network
twice for identical data

### the new endpoint

already existing:

```
GET /api/portfolio/risk
```

new:

```
POST /api/portfolio/what-if
```

the question it answers is *if i made this trade, what would my risk become?*

you send:

1. symbol ticker
2. "buy" or "sell"
3. quantity

and it:

- loads the real portfolio
- computes risk "before"
- makes the hypothetical portfolio
- computes risk "after"
- compares before vs after

‼️ nothing is saved to the database. purely a what-if

its a POST because it takes a body, not because it writes anything

```python
@router.post("/what-if")
```

### pydantic

pydantic is a python library for data validation. you define what shape you
expect incoming data to have — a class with typed fields — and pydantic checks
the real data against it, converts where that makes sense, and raises an error
where it doesnt

the shape a valid what-if request has to match:

```python
class WhatIfRequest(BaseModel):
    # 10 char str
    ticker: str = Field(min_length=1, max_length=10)

    # "buy" or "sell"
    action: Literal["buy", "sell"]

    # float greater than 0
    quantity: float = Field(gt=0)
```

which kills three of the five edge cases from the plan before any of my code
runs. `gt=0` means no zero-quantity trades and no negative ones, `Literal`
means no third action, `max_length` means no novel-length ticker reaching the
database

the two that pydantic cant catch are the ones that need to know the state of
the world — symbol not in the database (404) and selling more than you own
(400). those stay in the handler, because theyre not shape problems

‼️ this is the rule for anything that takes input. currently thats two places:

1. inputting a holding
2. inputting a what-if (here)

### then tests

same as milestone 6 — the degenerate cases are the whole point. sell down to
one holding, sell everything, buy something untracked

backend done
