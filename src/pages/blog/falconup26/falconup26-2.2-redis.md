---
title: "FalconUp 2026 - redis"
date: "2026-07-04"
description: "Swapping my hand-rolled cache for Redis, installing it on oc40, and throwing together a quick frontend"
dayNumber: 6
milestone: 2
milestoneTitle: "data source integration"
---

! clarification

milestone2 is data sources, milestone3 is database schema and stuff

right now data is being saved nowhere (live api calls + cache)

### redis

use redis for cache instead of writing your own cache from scratch

Redis: caching python library

- key-value store that lives outside of app
- survives server restarts and can be shared across multiple instances

```jsx
# Store a value (with expiration)

r.set("some_key", some_bytes, ex=60)   # ex=60 means expires in 60 seconds

# Retrieve a value

data = r.get("some_key")            # returns bytes, or None if expired/missing

# Delete a value

r.delete("some_key")
```

Pickle: python library

use alongside Redis

→ redis stores data in bytes

→ pickle converts bytes back into objects

```jsx
pickle.dumps(value) — converts your Python object into bytes before storing

pickle.loads(data) — converts bytes back into a Python object after retrieving
```

TTL: time to live  (cache terminology)

how many seconds until cache value thrown away

redis tests use `fakeredis` package lol

### trying out running redis

it all works!

now to install redis on the oc40 server:

first update apt:

```jsx
sudo apt update
```

install redis

```jsx
sudo apt install redis-server -y
```

config redis

sudo nano `/etc/redis/redis.conf` file

```jsx
bind 127.0.0.1 -::1

-- change to -->

bind 127.0.0.1 172.17.0.1

## tells redis to use docker ip
```

```jsx
protected-mode yes

-- change to -->

protected-mode no

## allows non localhost ips to work
```

enable redis:

```jsx
sudo systemctl enable redis-server
```

restart it:

```jsx
sudo systemctl restart redis-server
```

test:

```jsx
ubuntu@oc40:~$ redis-cli ping
PONG
```

nice!!

### make a quick frontend ui

![The first FalconUp frontend pulling live quote data](/blog-images/falconup26-2.2-redis/image.png)

nice!! 🎉
