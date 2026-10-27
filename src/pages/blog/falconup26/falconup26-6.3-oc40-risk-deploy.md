---
title: "FalconUp 2026 - deploying risk"
date: "2026-07-14"
description: "Milestone 6: no new tables, no new secrets — the first deploy in this project that was genuinely just the routine"
dayNumber: 18
milestone: 6
milestoneTitle: "portfolio risk"
---

no new migrations. regular steps

which is a sentence i could not have written for any previous milestone.
milestone 3 added tables, milestone 5 needed a production auth secret generated
on the box — every deploy so far had its own little surprise

this one doesnt, and the reason is the thing from the last post: risk math is
pure functions over data that already exists. holdings were already a table,
prices were already a table. the whole milestone adds computation, not state. so
theres nothing to migrate

### pull containers from ghcr

```jsx
cd ops
docker compose --env-file ../.env down
docker compose --env-file ../.env pull
docker compose --env-file ../.env up -d
```

### steps to host on oc40

```jsx
ssh ubuntu@oc40
git pull
```

check `.env` — nothing new to add this time, but still worth looking at, since
thats exactly the step i skipped before the milestone 5 deploy bit me

```jsx
cd backend
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/falconup uv run alembic upgrade head
```

still run the migration even with nothing to apply. it's a no-op when head is
already current, and running it unconditionally means i never have to remember
*whether* this particular milestone had one

```jsx
cd ../ops
docker compose -f compose.build.yaml down
docker compose -f compose.build.yaml up --build -d
```

and thats milestone 6. you can put in what you own and the app will tell you how
the pieces interact, not just whether each piece looks good on its own
