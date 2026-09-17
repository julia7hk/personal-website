---
title: "FalconUp 2026 - deploy milestone 2"
date: "2026-07-04"
description: "Shipping milestone 2 to oc40: ghcr images instead of building on the server, a PAT for docker login, and pull-to-deploy"
dayNumber: 7
milestone: 2
milestoneTitle: "data source integration"
---

merge pr to main

git pull on oc40

edit `.env` on oc40

```jsx
#-------------------------------------------------------------------------------
## 2b. redis

REDIS_URL=redis://172.17.0.1:6379/0

#-------------------------------------------------------------------------------
## 3. market data APIs (fill in the ones you use)

.
.
.

# https://fred.stlouisfed.org/docs/api/fred/ — macro data
FRED_API_KEY=<your-fred-api-key>
```

cd into `ops`

docker compose down

```jsx
ubuntu@oc40:~/_proj/falconup26/ops$ docker compose --env-file ../.env down
[+] down 4/4
 ✔ Container falconup-nginx     Removed        0.2s
 ✔ Container falconup-frontend  Removed        0.7s
 ✔ Container falconup-backend   Removed       10.2s
 ✔ Network falconup-40_falconup Removed        0.0s
ubuntu@oc40:~/_proj/falconup26/ops$
```

docker compose up (build locally on oc40)

```jsx
docker compose -f compose.build.yaml --env-file ../.env up -d --build
```

… ok this is taking a long time

.

.

.

### setting up GitHub Container Registry

1.

`.github/workflows/ci.yaml`

- builds docker image
- push to ghcr when merge to main

update it!

- remove jenkins stuff related to nc01 server
- etc

2.

create GitHub Personal Access Token (PAT)

1. GitHub → Settings → Developer settings → Personal access tokens.

    ‼️ necessary for oc40 to automatically allow docker to pull from ghcr

2. create token (classic) w/ read:packages perms

fine-grained token: newer, least privileges-type token but not completely there yet

doesnt support `read:packages` scope yet necessary for docker/ghcr

token (classic): classic token. but security risk because it has perms for everything and is quite problematic if leaked

```jsx
note: oc40-ghcr
expiration: none

select scopes: read:packages
```

1. log in on oc40 w/ newly created token

    ```jsx
    ubuntu@oc40:~/_proj/falconup26/ops$ echo <your-ghcr-token> | docker login ghcr.io -u julia7hk --password-stdin

    WARNING! Your credentials are stored unencrypted in '/home/ubuntu/.docker/config.json'.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/go/credential-store/

    Login Succeeded
    ubuntu@oc40:~/_proj/falconup26/ops$
    ubuntu@oc40:~/_proj/falconup26/ops$
    ```

1. push github `ci.yaml` changes to main

    all checks need to pass

    github → repo → packages

    [https://github.com/julia7hk?tab=packages](https://github.com/julia7hk?tab=packages)

    ![The frontend and backend packages published to ghcr](/blog-images/falconup26-2.3-deploy/image.png)

    2 containers: frontend, backend

    both run on oc40

    nginx routes to them correctly

2. pull the images from ghcr → oc40

```jsx
ubuntu@oc40:~/_proj/falconup26/ops$ docker compose --env-file ../.env pull
[+] pull 22/22
 ✔ Image nginx:latest                                Pulled      2.8s
 ✔ Image ghcr.io/julia7hk/falconup26/frontend:latest Pulled    106.5s
 ✔ Image ghcr.io/julia7hk/falconup26/backend:latest  Pulled     54.7s
ubuntu@oc40:~/_proj/falconup26/ops$
```

---

Once CI goes green,

on oc40:

```jsx
docker compose --env-file ../.env down
docker compose --env-file ../.env pull
docker compose --env-file ../.env up -d
```

nice !!!!! 🎉 🎉 🎉 🎉 🎉

![Milestone 2 live on falconup.julia7hk.com](/blog-images/falconup26-2.3-deploy/image%201.png)
