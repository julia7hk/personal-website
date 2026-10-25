---
title: "FalconUp 2026 - seed default symbols"
date: "2026-07-06"
description: "Seeding 16 symbols into the symbol table, then backfilling 5 years of price and macro history"
dayNumber: 10
milestone: 3
milestoneTitle: "database schema"
---

tables are empty. time to put something in them

### picking the 16 symbols

my holdings:

- QQQ, TQQQ, SOXL, NVDA

popular index ETFs:

- SPY (S&P 500), VOO (S&P 500 Vanguard), VTI (total market), DIA (Dow)

leveraged ETFs (need correct `leverage_factor`):

- SQQQ (3x inverse Nasdaq), SPXL (3x S&P 500)

big tech stocks (most likely first individual stocks):

- AAPL, MSFT, GOOGL, AMZN, META, TSLA

benchmarks (needed for beta calculations):

- SPY (already listed), used as the S&P 500 proxy

### `/backend/scripts/`

contains one-off scripts

### seed script

`/backend/scripts/seed.py`

puts the 16 symbols into the symbol table

the file contains the array of 16 symbols, so its literally just inserting
it into the table lol

```jsx
import asyncio

from sqlalchemy import text

from db import async_session

SYMBOLS = [
	{"ticker": "QQQ", "name": ...
]

async def main():
    async with async_session() as session:
        for sym in SYMBOLS:
            await session.execute(
                text("""
                    INSERT INTO symbol (ticker, name, type, sector, industry, leverage_factor)
                    VALUES (:ticker, :name, :type, :sector, :industry, :leverage_factor)
                    ON CONFLICT (ticker) DO NOTHING
                """),
                {
                    "ticker": sym["ticker"],
                    "name": sym["name"],
                    "type": sym["type"],
                    "sector": sym.get("sector"),
                    "industry": sym.get("industry"),
                    "leverage_factor": sym.get("leverage_factor", 1),
                },
            )
        await session.commit()
    print(f"Seeded {len(SYMBOLS)} symbols")

if __name__ == "__main__":
    asyncio.run(main())
```

`ON CONFLICT (ticker) DO NOTHING` is what makes this safe to run twice —
it leans on the unique constraint from the migration

```jsx
uv run python -m scripts.seed
```

it worked:

![All 16 seeded symbols in the symbol table, with ticker, name, type and sector filled in](/blog-images/falconup26-3.3-seed-symbols/image.png)

### backfill script

`/backend/scripts/backfill.py`

read the symbol table

fill `price_history` table with info for all those symbols

also backfill macro data (fred)

use `YFinanceProvider.get_history()` and `FredProvider.get_series_history()`
— the providers i already built in milestone 2, so no new api code

5 years of data ‼️

(adjustable with the script)

```jsx
async def backfill_prices(provider: YFinanceProvider) -> int:
    end = date.today()
    start = end - timedelta(days=YEARS * 365)
    rows_inserted = 0

    async with async_session() as session:
        result = await session.execute(text("SELECT id, ticker FROM symbol"))
        symbols = result.fetchall()

    for symbol_id, ticker in symbols:
    
    .
    .
    .
```

```jsx
async def backfill_macro(provider: FredProvider) -> int:
    end = date.today()
    start = end - timedelta(days=YEARS * 365)
    rows_inserted = 0

    for series_key, series_id in SERIES.items():
        print(f"  {series_key} ({series_id})...", end=" ", flush=True)
        data = provider.get_series_history(series_key, start, end)
        if not data:
            print("no data")
            continue

        async with async_session() as session:
        .
        .
        .
```

```jsx
async def main():

    yf_provider = YFinanceProvider()
    price_rows = await backfill_prices(yf_provider)

    fred_provider = FredProvider()
    macro_rows = await backfill_macro(fred_provider)

if __name__ == "__main__":
    asyncio.run(main())
```

run with

```jsx
uv run python -m scripts.backfill
```

<aside>
💡

fun fact: these two do the same thing but the `-m` flag and dot file path is more reliable for imports

```jsx
python scripts/backfill.py

python -m scripts.backfill
```

</aside>

uh oh

![Prices backfill succeeds with 20064 rows, then macro backfill crashes with AttributeError: 'str' object has no attribute 'toordinal'](/blog-images/falconup26-3.3-seed-symbols/image%201.png)

prices went fine — 1254 bars each, 20064 rows total — then macro blew up

little bug with macro backfill because dates are returned as strings vs. date objects

```jsx
"date": point["date"]
vs
"date": date.fromisoformat(point["date"])
```

`asyncpg` wants a real `date` object for a `DATE` column and FRED hands back
`"2021-07-06"`, so it was calling `.toordinal()` on a plain string

rerun `uv run python -m scripts.backfill`

![macro_history filled with fed_funds_rate rows going back to 2021, price_history at 4.4M](/blog-images/falconup26-3.3-seed-symbols/image%202.png)

nice !!!
