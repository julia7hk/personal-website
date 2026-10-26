---
title: "FalconUp 2026 - auth"
date: "2026-07-10"
description: "Milestone 5: everyone could see everyone else's holdings, so I learned how auth actually works and wired up Better Auth + Prisma"
dayNumber: 14
milestone: 5
milestoneTitle: "portfolio input"
---

so all the crud features of adding a portfolio holding work

but if one person inputs a holding, literally everyone else on the site can see
it too upon opening the website 💀

on my computer, i go to [falconup.julia7hk.com](http://falconup.julia7hk.com)
and add a holding for QQQ

on my phone, i open [falconup.julia7hk.com](http://falconup.julia7hk.com) and
its still there.. thats not good

its good that it persists and doesnt get lost tho !

lets create a proper auth !!

---

### learning

where do i even begin

**hosted services:** run the auth service for you — e.g. supabase, firebase

**self hosted libraries:** you set up your own auth — e.g. better-auth,
auth.js/nextauth

#### how auth works:

things you need:

1. who is who? (authentication)
2. remember that they proved their identity (sessions)
3. separate each person's stuff (multi-tenancy)

**step 1: passwords**

- always store jumbled string w/ hash function (bcrypt)
    - when user enters password, put input into the same hash function and see
      if they match

**step 2: sessions**

- session token, like a wristband
    - proves identity, give wristband
    - you can check that they have a wristband
    - wristband expires at the end of time period
- session stored in a table (token, user id, expires at)
- every request looks up token

**step 3: cookies**

- how the browser holds the wristband

**step 4: multi tenancy**

- every row, every query has user_id

better auth does all steps above for you — hash, sessions, tokens, cookies, and
the database tables

im going to go with Better Auth

also integrate prisma (i need it eventually, might as well do now)

<u>Prisma:</u> ORM for javascript

better SQL queries, autocomplete, type safety, generate client

---

### actual building

step 1: alembic migration for better auth tables

step 2: prisma — all in frontend ?

step 3: better auth

```jsx
npm install better-auth
```

step 4: fastapi session bridge

step 5: infra

- nginx routing
- dockerfile
- ci
- env vars

### testing

```jsx
cd backend

DATABASE_URL=postgresql://postgres:postgres@localhost:5432/falconup uv run uvicorn main:app --reload --port 40401
```

```jsx
cd frontend

npm run dev
```

debugging — two bugs fixed:

- cookie parsing (better auth token url encoding vs db storing raw token)
- local dev routing (next.js `/api/auth/*` going to fastapi instead of better
  auth)

### frontend edits

- separate landing page info vs. logged in info
- profile page

now the holdings are actually *mine*
