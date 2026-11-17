---
title: "FalconUp 2026 - explainer dropdown"
date: "2026-07-16"
description: "Milestone 5: a SignalBreakdown dropdown that shows which indicators are actually driving a buy/hold/sell call — no new math, just the reasoning made visible"
dayNumber: 20
milestone: 5
milestoneTitle: "portfolio input"
---

finally adding the dropdown that explains the buy/hold/sell rating

up to now the composite card showed a verdict and a confidence number and
nothing else. "BUY, 6% confidence" is not an explanation, its an assertion — and
a 6% confidence assertion especially needs to show its work

### the changes

new `SignalBreakdown` component, used in two places:

- the homepage's `CompositeCard`
- the dashboard's `HoldingSignalPanel`

no new computations. the contributions were already in the api response from
milestone 4 — this is purely UI render over data that was already being thrown
away

### per-indicator breakdown

each of the 9 indicators gets a bar showing its signed contribution to the
composite, sorted by **absolute** contribution

sorting by absolute value means "what is driving this the hardest" floats to the
top regardless of whether its bullish or bearish. sorting signed would have put
all the green at one end and all the red at the other, which looks tidier and
tells you less

![QQQ composite card expanded — BUY at 6% confidence, +0.152, with the nine indicator contribution bars sorted by magnitude underneath](/blog-images/falconup26-5.4-signal-breakdown/image.png)

QQQ is a good example of why this was worth building. 5 indicators bullish, 4
bearish, SMA 50/200 carrying most of the weight at +0.130 and MACD pulling back
at −0.074. the 6% confidence isnt the model being useless, its the model
correctly reporting that the indicators disagree

### on the dashboard

`HoldingSignalPanel` is the dashboard-side wrapper. the dashboard now fetches
the full indicator payload rather than just the composite, since
`SignalBreakdown` needs the per-indicator contributions

expansion state changed from a boolean-ish

```jsx
useState<string | null>
```

to a toggle keyed on ticker

```jsx
expandedTicker === h.ticker ? null : h.ticker
```

so only one row is open at a time. with the breakdown being ~9 bars plus
explanation text, every row expanded at once is a wall

### the disclaimer

‼️ the signal reflects THE TICKER, NOT YOUR POSITION SPECIFICALLY

this matters more now that the breakdown sits on the dashboard next to your
actual holdings. the composite doesnt know your cost basis, your position size,
or when you bought. it says what the indicators say about the symbol — the
personalization is which symbols you see, not what the math is computed over
