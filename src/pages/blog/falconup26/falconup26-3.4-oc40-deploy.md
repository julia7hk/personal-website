---
title: "FalconUp 2026 - building on oc40"
date: "2026-07-07"
description: "A pre-push git hook, then running migrations and both scripts on the oc40 server to deploy milestone 3"
dayNumber: 11
milestone: 3
milestoneTitle: "database schema"
---

update ui changes to see the data that i built

![The FalconUp symbol catalog grouped into index ETFs, leveraged ETFs and stocks](/blog-images/falconup26-3.4-oc40-deploy/image.png)

![AAPL price history chart over 250 trading days, plus the macro indicator tiles](/blog-images/falconup26-3.4-oc40-deploy/image%201.png)

### pre-push hook

i pushed these changes to remote but then github ci flagged down some errors
because the tests + linters didnt pass.

fixed those errors

but also added pre-push git hook

(just a quick bash script)

`falconup26/.git/hooks/pre-push`

```jsx
#!/bin/sh
set -e

ROOT="$(git rev-parse --show-toplevel)"

echo "Running backend tests..."
cd "$ROOT/backend"
uv run pytest --tb=short -q

echo "Running frontend lint..."
cd "$ROOT/frontend"
npm run lint

echo "All checks passed."
```

`set -e` is the important line — without it the script keeps going after a
failing command and happily reports "All checks passed"

this runs the same checks ci does, just before the push instead of after, so
i find out in 30 seconds instead of waiting on a red x

pushed, merged, all good now

---

### set up changes on oc40

1. git pull

```jsx
git reset --hard origin/main
```

2. install uv

```jsx
curl -LsSf https://astral.sh/uv/install.sh | sh
```

3. install direnv

```jsx
sudo apt install direnv
```

```jsx
echo 'eval "$(direnv hook bash)"' >> ~/.bashrc
```

log out and log back in  OR  `source ~/.bashrc`

type `direnv allow` for falconup

4a. run alembic migrations to create the tables

‼️ have to override the database url temporarily to use localhost instead of docker ip

since this is supposed to run on the oc40 bare metal instead of in one of the docker containers

```jsx
uv run alembic upgrade head

->

DATABASE_URL=postgresql://postgres:postgres@localhost:5432/falconup uv run alembic upgrade head
```

creates all 4 tables

4b. run seed script to fill initial 16 symbols

```jsx
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/falconup uv run python -m scripts.seed
```

4c. run backfill script to get 5 years of data for the 16 symbols

```jsx
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/falconup uv run python -m scripts.backfill
```

5. rebuild docker containers

`cd ops`

```jsx
docker compose -f compose.build.yaml down
```

```jsx
docker compose -f compose.build.yaml up --build -d
```

check `falconup.julia7hk.com`

![falconup.julia7hk.com live, serving the 16-symbol catalog out of Postgres](/blog-images/falconup26-3.4-oc40-deploy/image%202.png)

nice!!!

the data is coming out of the database now instead of a live yfinance call,
which is the whole point of this milestone

---

### side tangent:

switch to raw sql to do alembic migrations instead of using sqlalchemy

why?

just cuz :P
