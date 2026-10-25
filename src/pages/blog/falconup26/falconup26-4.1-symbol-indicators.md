---
title: "FalconUp 2026 - symbol calculations"
date: "2026-07-08"
description: "Milestone 4: seven technical indicators, a weighted composite score, and discovering the composite says HOLD on everything"
dayNumber: 12
milestone: 4
milestoneTitle: "symbol indicator calculations"
---

### finance background:

standard technical analysis indicators that traders and investors have used for decades

1. RSI → overbought vs oversold?
2. MACD → is trend speeding up vs slowing down
3. Bollinger Bands → how volatile is the stock / how much variation is there in average
4. SMA crossover → long term trend? 50-day avg vs. 200-day avg
5. ATR → how much this stock usually moves in a day
6. Beta → how much this stock moves relative to market
7. Sharpe ratio → am I being compensated for the risk im taking? aka returns vs. volatility

covers the following major categories

| **Category** | **Meaning** | **Indicators** |
| --- | --- | --- |
| Momentum | up/down how fast | RSI, MACD |
| Trend | long term direction | SMA crossover |
| Volatility | how much swing | Bollinger, ATR |
| Risk | how risky relative to market | Beta, Sharpe |

start computing these formulas for each of the 16 symbols we scraped data on for the last 5 years

all of this reads out of `price_history` and `macro_history` — no network
calls in the math layer, which was the whole point of milestone 3

### start building

sections:

- `indicators/math.py` math formulas
- `indicators/composite.py` composite of 7 indicators
- `indicators/models.py` dataclasses
- `routers/indicators.py` api

order of milestone 4:

1. make the dataclasses / models - define the shape
2. make the math functions / formulas
3. make the composite / combine formula results
    - normalize all 7 indicators to a [-1, 1] scale
    - combine with weighted sum
    - classify buy/sell/hold
4. make tests for math and composite
5. make the router/api and connect it to backend

### the composite

each indicator gets mapped onto the same [-1, +1] scale, where +1 is
strongly bullish and -1 is strongly bearish

```jsx
# Weights must sum to 1.0
WEIGHTS: dict[str, float] = {
    "rsi": 0.15,
    "macd": 0.15,
    "bollinger": 0.10,
    "sma_crossover": 0.15,
    "atr": 0.10,
    "beta": 0.15,
    "sharpe": 0.20,
}
```

RSI is the easy one to picture — 30 is oversold so that's bullish, 70 is
overbought so that's bearish, 50 is nothing:

```jsx
# RSI 30 -> +1 (oversold/bullish), 50 -> 0, 70 -> -1 (overbought/bearish)
return max(-1.0, min(1.0, (50 - result.value) / 20))
```

then multiply each normalized value by its weight, sum them, and that's the
composite score

### test whats been built!

start backend

```jsx
uv run uvicorn main:app --reload --port 40401
```

hit indicator api

```jsx
➜  ~ curl localhost:40401/api/symbols/QQQ/indicators

{"ticker":"QQQ","computed_at":"2026-07-08","data_points":406,"indicators":{"rsi":{"value":48.25,"signal":"neutral"},"macd":{"macd_line":3.4286,"signal_line":5.5481,"histogram":-2.1195,"signal":"bearish"},"bollinger":{"width":0.0744,"upper":747.48,"lower":693.86,"signal":"neutral"},"atr":{"value":16.5372,"atr_percent":2.32},"beta":{"value":1.23,"interpretation":"more volatile than market"},"sharpe":{"value":0.98,"risk_free_rate":3.63,"interpretation":"positive but below-average risk-adjusted return"},"sma_crossover":{"sma_50":708.51,"sma_200":633.49,"crossover_type":"none","days_since_cross":null}},"composite":{"score":-0.0766,"signal":"hold","confidence":0.11,"contributions":{"rsi":0.0131,"macd":-0.0927,"bollinger":0.0,"sma_crossover":0.0,"atr":-0.05,"beta":-0.045,"sharpe":0.098}}}%
```

nice!!  theres rsi, macd, etc.

update ui too

![SQQQ technical indicator panel — HOLD at 11% confidence, with the seven indicator tiles below it](/blog-images/falconup26-4.1-symbol-indicators/image.png)

confidence:  based on how much the 7 indicators agree with each other

### new problem:  composite normalization doesnt work that well

says HOLD with like 10% confidence on all of them

look at the contributions in that QQQ response — the biggest one is macd at
-0.09 and everything else is basically zero. bollinger and sma_crossover are
*literally* 0.0

useful information from the indicators is getting lost because they all get normalized with a score

and SQQQ in the screenshot is a **3x inverse** ETF in a year where it went
from $92 to $35 — "hold, 11% confidence" is not a real answer

rework the system!!
