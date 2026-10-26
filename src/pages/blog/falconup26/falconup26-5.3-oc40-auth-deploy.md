---
title: "FalconUp 2026 - hosting on oc40"
date: "2026-07-11"
description: "Milestone 5: deploying the auth branch to the Oracle box turned out to need one more step — a production Better Auth secret"
dayNumber: 15
milestone: 5
milestoneTitle: "portfolio input"
---

i thought it would be the same process but no, theres a little more to it after
pushing because i need a better auth key

---

- auth pr merged to main

- ssh ubuntu
- git pull

- edit `.env` on oc40

```jsx
#-------------------------------------------------------------------------------
## 2a. auth (Better Auth)

BETTER_AUTH_SECRET=
NEXT_PUBLIC_BETTER_AUTH_URL=https://falconup.julia7hk.com

#-------------------------------------------------------------------------------
```

- generate new better auth secret key

```jsx
openssl rand -base64 32
```

the secret has to be generated *on the server* and kept out of the repo — it's
what signs every session cookie, so if it leaks, anyone can forge a session. the
local dev value and the production value are deliberately different

everything else was the same as the milestone 3 deploy: pull, rebuild, restart
