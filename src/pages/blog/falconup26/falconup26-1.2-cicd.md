---
title: "FalconUp 2026 - ci/cd"
date: "2026-07-02"
description: "GitHub Actions CI on every push, and planning the CD path through ghcr and Jenkins"
dayNumber: 2
milestone: 1
milestoneTitle: "project foundation"
---

# CI/CD

### what is Github Actions CI/CD

CI/CD:  Continuous Integration, Continuous Deployment

CI → automatically check/test code every push

CD → automatically deploy code when it passes CI

Github Actions:  Github’s automation system

when something is pushed,

github server:

- makes new ubuntu vm
- clones your repo
- does steps outlined in `.github/workflows/ci.yaml`

`.github/workflows/ci.yaml`  commonly contains:

- `npm run build`
- `npm run lint`
- `uv run pytest`

pytest:  python test framework. write test cases.

---

### the pr itself:

create `.github/workflows/ci.yml`

```jsx
name: CI

on:
	push:
		...
	pull_request:
		...

jobs:
	backend:
		...
	
	frontend:
		...
```

!! this completes the CI part, but not the CD part

dont want to automatically deploy to nc01 just yet

omg it works yay!

![CI passing on GitHub Actions](/blog-images/falconup26-1.2-cicd/image.png)

### THE DOCKER CI/CD PART

build docker container on local

`compose.build.yaml` file

push container to github container registry (ghcr)

nc01 server pulls image from ghcr

`compose.yaml`

docker compose down, up

done through a jenkins job
