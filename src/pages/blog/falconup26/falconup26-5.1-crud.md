---
title: "FalconUp 2026 - CRUD"
date: "2026-07-09"
description: "Milestone 5 kickoff: portfolio holdings you can actually create, read, update, and delete — backend first, then frontend"
dayNumber: 13
milestone: 5
milestoneTitle: "portfolio input"
---

milestone 5: creating a way on the app to input your stuff

up until now the whole app has been read-only. symbols get seeded, prices get
scraped, indicators get computed — but there is no way for *me* to tell the app
what i actually own

that is the thing that unlocks every milestone afterwards. portfolio-level
analytics, allocation breakdowns, risk on *your* holdings instead of on a
generic list of 16 tickers — none of it exists until there are holdings to
analyze

so: CRUD ‼️

### order of operations

start with backend first, then frontend

frontend works off backend — if the endpoints aren't shaped right, the UI gets
built against a shape that has to change anyway. do the boring part first

- `/router/portfolio.py` portfolio api endpoints

the `portfolio_holding` table already exists from milestone 3, so this is
mostly wiring: create a holding, list holdings, update one, delete one. the
four verbs

### what this sets up

once holdings go in, the indicator work from milestone 4 stops being abstract.
instead of "QQQ says hold at 11% confidence" it becomes "your position in QQQ
says hold" — and eventually, aggregated across everything you own

one problem though, which i only noticed after it worked
