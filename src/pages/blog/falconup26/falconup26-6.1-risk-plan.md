---
title: "FalconUp 2026 - portfolio risk plan"
date: "2026-07-12"
description: "Milestone 6 kickoff: per-symbol indicators tell you when, portfolio risk tells you what you're actually holding — planning the metrics before writing any of them"
dayNumber: 16
milestone: 6
milestoneTitle: "portfolio risk"
---

milestone 6: analyzing the risk of the portfolio you just put in

### what do i have already?

per symbol indicators. from milestone 4 — RSI, moving averages, the whole
per-ticker signal thing

### what do i want?

i want to analyze the risk of my portfolio *when i input it*

see how all the individual symbols in my portfolio interact with each other and
whether thats good or not

because here is the thing i kept running into: the app can tell me QQQ says hold
and TQQQ says hold and SOXL says hold, and every single one of those can be
correct, and the portfolio can still be a disaster. three green lights on three
things that all fall over at the same time is not three signals. its one signal
printed three times

the framing that made it click:

> symbol indicators = **timing** signal
>
> portfolio risk = **structural** signal

timing is "is now a good moment". structure is "what happens to me if im wrong".
completely different question, needs completely different math

### what milestone 6 adds

- concentration in sector
- correlation between each other's movements
- effective leverage (effectively 3x or 2x or what compared to movement)
- portfolio beta
- max drawdown estimate
- stress scenarios
- risk grade

### how does it add these things?

each one maps to an actual established metric, which is good, because it means
im not inventing numbers

- <u>Herfindahl Index (HHI):</u> measures concentration
- <u>Correlation Matrix:</u> how similarly two stocks move day-to-day
- <u>Weighted Beta:</u> how stock moves against S&P500
- <u>Leverage Aggregation:</u> is your portfolio effectively 3x or 2x or what

### the two subjective ones

the four above are just math. you put holdings in, a number comes out, nobody
has to agree with me about it

the other two are not like that, and i want to be honest about which is which

**risk grade:** subjective. its me picking thresholds and deciding that HHI
above some number is bad. so the rule i gave myself is that the grade always
ships with the reasoning behind it — not just "C", but *why* C. a letter with no
explanation is me asking you to trust my taste, and theres no reason to. a
letter with the breakdown underneath it is something you can disagree with

**stress scenarios:** also subjective, but theres a cheat available. instead of
making up a scenario like "what if the market drops 30%", use *historical
scenarios* — real periods that actually happened, with real numbers. covid
march 2020, 2022 rate hikes, etc. then the scenario isnt my opinion, its a date
range

next: actually writing the math
