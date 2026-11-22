---
title: "FalconUp 2026 - what-if frontend"
date: "2026-07-19"
description: "Milestone 7, finished: types, a WhatIfPanel that renders the before/after diff as a table, and a portfolio grade that finally moves when you touch it"
dayNumber: 23
milestone: 7
milestoneTitle: "what-if portfolio"
---

backend is done, so the rest of milestone 7 is making it visible

three steps, and theyre small compared to the refactor that came before them

### 1. update types

`src/types.ts` gets the three new shapes:

```ts
export type WhatIfRequest = {
  ticker: string;
  action: "buy" | "sell";
  quantity: number;
};

export type WhatIfDiffEntry = {
  before: number | null;
  after: number | null;
  delta: number | null;
  direction: "improved" | "worsened" | "unchanged" | "unavailable";
  // Only present on the risk_grade entry.
  before_grade?: string | null;
  after_grade?: string | null;
};

export type WhatIfResponse = {
  trade: WhatIfRequest;
  // before/after reuse the RiskData shape (same _compute_risk_metrics payload).
  before: RiskData;
  after: RiskData;
  diff: Record<string, WhatIfDiffEntry>;
  disclaimer: string;
};
```

`WhatIfRequest` is just the pydantic model from the last post, restated in
typescript. the two sides have to agree on the shape and theres no codegen in
this project, so it gets typed twice

the part thats actually new is `WhatIfDiffEntry`. the backend doesnt just hand
back two risk reports and leave the frontend to subtract them — it sends the
comparison already made. `before`, `after`, `delta`, and a `direction` that
says what the change *means*

that last field is the one that matters, because the sign of the delta doesnt
tell you whether its good news. concentration going up is bad. beta going down
is usually good. drawdown is negative already, so a number getting larger is an
improvement. working that out per-metric in the component would mean
re-encoding risk semantics in the ui, where theyd drift from the engine the
first time i changed a weight. so `direction` is computed where the math lives
and the component just reads it

`"unavailable"` is the fourth direction, and thats the one-holding correlation
case from the plan finally surfacing as a ui state instead of a crash

note also that `before` and `after` are both plain `RiskData` — the same shape
`/api/portfolio/risk` already returns. the dashboard already knows how to
render that. two of them is not a new rendering problem

### 2. write the `WhatIfPanel` component

inputs are the three fields from the request type — symbol, buy/sell toggle,
shares — and a simulate button. then the response renders as a table, one row
per metric, with now → after → change

![The dashboard with the What-If Analysis panel expanded, simulating a buy of 40 shares of VOO and showing the metric-by-metric before/after diff](/blog-images/falconup26-7.3-what-if-frontend/image.png)

the screenshot is a 3-holding portfolio thats failing — F, 34.9 out of 100,
"extreme concentration, leverage, and/or correlation". which is fair. its NVDA,
QQQ and TQQQ, so its one bet on one sector with a 3x fund stapled to it

simulate buying 40 shares of VOO and the grade goes F (35) → D (49)

the rest of the table is why. leverage 1.37x → 1.07x, beta 1.77 → 1.13, max
drawdown -46.8% → -28.7%, all marked better. and concentration 0.619 → 0.677,
marked **worse**

that last row is the whole feature working. buying a broad market index fund
makes the portfolio safer on three metrics and *more* concentrated on the
fourth, because 40 shares of VOO is a large enough position that it becomes the
new dominant holding. the HHI goes up even though the risk went down

thats not something i would have predicted by reading the grade. and its not
something the milestone 6 page could ever have shown, because it only ever had
one portfolio to describe

the two grade cards at the bottom are the existing risk grade component
rendered twice, now vs after trade, which is the payoff of `before` and `after`
being the same shape

and the footer is doing real work: *simulation only — your portfolio is
unchanged. educational analysis, not financial advice.* the backend never
writes anything, but the user cant see that, so the ui has to say it

### 3. put it in the dashboard

below the holdings list, collapsible, collapsed by default

it goes under holdings because thats the order of the question. you look at
what you own, then you ask what if. and its collapsed because most visits
arent a what-if — the panel shouldnt push the risk report off the screen for
everyone who just wanted to check the grade

### milestone 7 done

the thing i said at the start of this milestone was that the grade should stop
being a verdict and become a dial. F → D by buying one index fund is that,
literally — and the concentration row going the other way is the part that
makes it worth building instead of just useful
