---
title: "FalconUp 2026 - create the base tables"
date: "2026-07-06"
description: "Milestone 3 kickoff: four alembic migrations for symbol, portfolio_holding, price_history, and macro_history"
dayNumber: 8
milestone: 3
milestoneTitle: "database schema"
---

milestone 3 is where the data finally stops being ephemeral

right now everything is live api calls + redis cache. nothing is saved anywhere

### what i want to store

- daily prices
- computed indicators
- sector/industry data

mid day prices stay in redis (short TTL, throwaway)

the architecture:

```jsx
yfinance (external)
  -> daily job backfills OHLCV into Postgres (source of truth)
  -> Redis still handles live quotes (short TTL, ephemeral)
```

the indicator engine should read from **Postgres, not yfinance**

→ deterministic, testable, no network dependency for calculations

! the daily job itself maps to milestone 8 (data pipeline). the schema
foundation goes here in milestone 3

so:

phase 1 (milestone 3):  schema + seed data

phase 2 (milestone 8): daily job to keep filling data

! bc a daily job only matters when the rest of the app works

---

### migrations!

use alembic (python database migration tool)

create alembic migrations + orm models

```jsx
cd backend

uv run alembic revision -m "create symbol table"
```

this generates a file in `/backend/alembic/versions`

edit the file

```jsx
def upgrade() -> None:
    op.create_table(
        "symbol",
        sa.Column("id", sa.Integer, primary_key=True),
        sa.Column("ticker", sa.Text, nullable=False, unique=True),
        sa.Column("name", sa.Text),
        sa.Column("type", sa.Text, nullable=False),  # 'etf' or 'stock'
        sa.Column("sector", sa.Text),
        sa.Column("industry", sa.Text),
        sa.Column("leverage_factor", sa.Numeric, nullable=False, server_default="1"),
        sa.Column("created_at", sa.DateTime(timezone=True), nullable=False, server_default=sa.func.now()),
        sa.Column("updated_at", sa.DateTime(timezone=True), nullable=False, server_default=sa.func.now()),
    )

def downgrade() -> None:
    op.drop_table("symbol")
```

do this 4 times total for 4 tables

1. symbol
2. portfolio_holding
3. price_history
4. macro_history

more tables to come in the future too !!

### the other three

`portfolio_holding` — what i actually own, foreign key back to symbol

```jsx
sa.Column("symbol_id", sa.Integer, sa.ForeignKey("symbol.id", ondelete="CASCADE"), nullable=False),
sa.Column("shares", sa.Numeric, nullable=False),
sa.Column("avg_cost", sa.Numeric, nullable=False),
```

`price_history` — one row per symbol per trading day (the OHLCV from milestone 2)

```jsx
sa.Column("date", sa.Date, nullable=False),
sa.Column("open", sa.Numeric, nullable=False),
sa.Column("high", sa.Numeric, nullable=False),
sa.Column("low", sa.Numeric, nullable=False),
sa.Column("close", sa.Numeric, nullable=False),
sa.Column("volume", sa.BigInteger, nullable=False),
sa.UniqueConstraint("symbol_id", "date", name="uq_price_history_symbol_date"),
```

the unique constraint is doing real work here — it means i can re-run the
backfill as many times as i want and never get duplicate rows

`macro_history` — same idea but keyed by series name instead of symbol

```jsx
sa.Column("series", sa.Text, nullable=False),  # fed_funds, vix, treasury_2y, treasury_10y, etc.
sa.Column("date", sa.Date, nullable=False),
sa.Column("value", sa.Numeric, nullable=False),
sa.UniqueConstraint("series", "date", name="uq_macro_history_series_date"),
```

both history tables get an index on `(symbol_id, date)` / `(series, date)`
since that's how every query is going to look them up

### apply the migrations

this will actually create the tables

```jsx
uv run alembic upgrade head
```

```jsx
➜  backend git:(main) ✗ uv run alembic upgrade head
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> b1ad558c300c, create symbol table
INFO  [alembic.runtime.migration] Running upgrade b1ad558c300c -> 92d3e069a43b, create portfolio_holding table
INFO  [alembic.runtime.migration] Running upgrade 92d3e069a43b -> 38cf1f8621f7, create price_history table
INFO  [alembic.runtime.migration] Running upgrade 38cf1f8621f7 -> bc5377847070, create macro_history table
➜  backend git:(main) ✗
```

each migration chains off the one before it (`down_revision`), so alembic
knows the order to run them in — and `alembic_version` tracks where the
database currently is

yay! 🎉

![All four tables plus alembic_version showing up in the falconup database](/blog-images/falconup26-3.1-base-tables/image.png)
