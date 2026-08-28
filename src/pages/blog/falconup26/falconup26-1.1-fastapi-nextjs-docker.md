---
title: "FalconUp 2026 - FastAPI, Next.js, Docker"
date: "2026-06-24"
description: "Setting up FastAPI + Uvicorn, scaffolding Next.js, and containerizing everything with Docker Compose"
dayNumber: 1
milestone: 1
milestoneTitle: "project foundation"
---

last dev section was **set up env, uv, alembic**

this time my goal is to set up ***FastAPI and Next.js***

i also got to **docker** today!!

well, its shifted into the next day because midnight has passed

---

### FastAPI explanation

API:  set of URLs that return data (not URLs that return web pages)

e.g.

```jsx
input:
GET /api/portfolio/123

output:
{"symbol": "QQQ", "shares": 10, "risk_score": 3.2}
```

FastAPI:  python library. tool for building APIs. helps define those URLs

- manually handles HTTP requests so you dont have to manually parse, care about Content-Type: application/json header, etc.
- auto-generates API docs **(Swagger docs)** !!

there are really only two concepts:

- The app object (`app = FastAPI()`)
- Route handlers (`@app.get(...)`)
    
    decorated functions that say "when this URL is hit, run ___"
    

Uvicorn:  python library. HTTP server

- Uvicorn — *listen on a port for incoming network requests* and forward them to FastAPI app.
- FastAPI — defines what to do with those requests.

---

### FastAPI PR (#2):

- uv already created a virtual env
- uv installed all dependencies (FastAPI, Uvicorn, SQLAlchemy, etc.) based on pyproject.toml

- [`main.py`](http://main.py) within /backend is the starting point for all backend stuff
- edit [`main.py`](http://main.py) to have a quick fastAPI integration:

```jsx
# main.py

from fastapi import FastAPI

app = FastAPI(title="FalconUp", version="0.1.0")

@app.get("/health")
async def health():
    return {"status": "ok"}

```

### TO RUN:

```jsx
cd backend
```

```jsx
uv run uvicorn main:app --reload --port 40401
```

- uv run — "use my virtual environment to run..."
- uvicorn — "...the Uvicorn server..."
- main:app — "...serving the app object from [main.py](http://main.py/)..."
- -port 40401 — "...listening on port 40401"

---

# NEXTJS

- run npx create-next-app
    - typescript, tailwindcss
    - app router (modern routing system)

---

# DOCKER

### why docker?

to run the app, you need

- python
- postgres
- nodejs
- etc etc

put it all in a lightweight env called a container 😁

docker image:  blueprint for starting up a container

Docker Engine:  core tool that builds images / runs containers

Docker Compose:  add-on that reads `docker-compose.yaml`  and orchestrates containers

this app has multiple containers that need to run (e.g. backend, frontend, nginx)

containers are isolated

but! they are connected together by the same `falconup` network

`compose.yaml`

```jsx
name: falconup-40
services:
  # One-shot DB migration. Runs `alembic upgrade head` and exits; the app
  # services wait for it to complete (service_completed_successfully) so they
  # never start against a schema older than the code.
  migrate:
    container_name: falconup-migrate
    image: ghcr.io/julia7hk/falconup26/backend:latest
    restart: "no"
    env_file:
      - ../.env
    command:
      - uv
      - run
      - alembic
      - upgrade
      - head
    **networks:
      - falconup**

  nginx:
    container_name: falconup-nginx
    image: nginx:latest
    restart: always
    ports:
      - "80:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/log:/var/log/nginx
    depends_on:
      - frontend
      - backend
    **networks:
      - falconup**

  backend:
    container_name: falconup-backend
    image: ghcr.io/julia7hk/falconup26/backend:latest
    restart: always
    env_file:
      - ../.env
    command:
      - uv
      - run
      - uvicorn
      - main:app
      - --host
      - "0.0.0.0"
      - --port
      - "40401"
    depends_on:
      migrate:
        condition: service_completed_successfully
    **networks:
      - falconup**

  frontend:
    container_name: falconup-frontend
    image: ghcr.io/julia7hk/falconup26/frontend:latest
    restart: always
    env_file:
      - ../.env
    depends_on:
      migrate:
        condition: service_completed_successfully
    **networks:
      - falconup**

**networks:
  falconup:
    driver: bridge**
```

---

### the actual pr

- the docker stuff lives in the  `ops/` folder

we’re going to have 3 key files:

- `docker-compose.yaml`

```jsx
volumes:                
  - pgdata:/var/lib/postgresql/data
```

- `backend/Dockerfile`
- `frontend/Dockerfile`

and additional

- `backend/.dockerignore`
- `frontend/.dockerignore`

---

### how it works / what happens

- `docker-compose.yaml`  reads `.env` for keys
- backend and frontend apps connected within container
- `docker-compose.yaml`  specifies data lives outside container on actual machine

1.

```jsx
docker compose up --build
```

run this in the `ops/` folder or wherever has the `docker-compose.yaml` file

create a container for the very first time

**if code changes, must run this command again to use latest code**

2. 

after building the first time,

```jsx
docker compose up
```

restarts containers

3. 

```jsx
docker compose down
```

stop image
