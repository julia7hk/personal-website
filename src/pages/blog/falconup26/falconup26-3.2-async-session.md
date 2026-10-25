---
title: "FalconUp 2026 - SQLAlchemy async session"
date: "2026-07-06"
description: "What an async session actually is, and the db.py that connects FastAPI to Postgres"
dayNumber: 9
milestone: 3
milestoneTitle: "database schema"
---

### what the heck does that mean???

tables exist now but nothing in the app can talk to them yet

the three pieces:

postgres → database tables thru alembic migrations

python/fastapi → code logic that reads/writes to postgres database

SQLAlchemy → bridge between postgres + python

### session

session: like an open conversation w/ the database

- handles multiple queries in one session
- managed by SQLAlchemy

### async

everything in fastapi is `async`/`await` already, so the database layer has
to be async too — otherwise a slow query blocks the whole event loop and
every other request waits behind it

that means the `asyncpg` driver instead of the default sync one

### actual code

create `db.py`

1. reads `DATABASE_URL` from env
2. swaps `postgresql://` to `postgresql+asyncpg://`
3. creates the async engine (connection pool to Postgres)
4. exports a session factory that the routes will use

```jsx
import os

from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker, create_async_engine

_raw_url = os.environ.get("DATABASE_URL", "")

# Defer engine creation so importing db.py doesn't crash in environments
# without DATABASE_URL (e.g. CI test collection, frontend-only builds).
engine = None
async_session = None

if _raw_url:
    # asyncpg requires the +asyncpg driver suffix
    _async_url = _raw_url.replace("postgresql://", "postgresql+asyncpg://", 1)
    engine = create_async_engine(_async_url, pool_pre_ping=True)
    async_session = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
```

the deferred engine thing bit me first — if you create the engine at import
time, CI blows up just collecting tests, because there's no `DATABASE_URL`
in that environment and nothing is even trying to hit the database

then the fastapi dependency:

```jsx
async def get_session():
    """FastAPI dependency — yields an async database session."""
    if async_session is None:
        raise RuntimeError("DATABASE_URL environment variable is not set")
    async with async_session() as session:
        yield session
```

`async with` means the session closes itself when the request is done, even
if the route raises

now every route can just take `session: AsyncSession = Depends(get_session)`
and it gets a live connection

bridge built
