---
title: "FalconUp 2026 - oracle"
date: "2026-07-02"
description: "Pivoting to my own Oracle Cloud ARM box (oc40): docker, postgres, and a firewall fight with Docker's bridge network"
dayNumber: 3
milestone: 1
milestoneTitle: "project foundation"
---

pivot off of daddy’s nc01 server

i was able to get another Seoul Ampere on Oracle Cloud

- Ampere chip
- ARM64 processor
    - NOT AMD 64 (aka x86_64)

‼️ convenient that ampere and mac m1 chips are both ARM

→ same docker images will build on both local and server !! :D

to get in right now:

```jsx
ssh ubuntu@<oc40-public-ip>
```

resolve this

edit `/etc/hosts` 

```jsx
sudo nano /etc/hosts
```

you should have to input ur macbook password bc sudo command

**after ssh setup is complete:**

```jsx
ssh ubuntu@oc40
```

### to install docker on oc40

step1

```jsx
curl -fsSL https://get.docker.com | sudo sh
```

[`https://get.docker.com`](https://get.docker.com) is a quick install script created by docker 

step2

docker commands require `sudo` by default

(`docker compose up` vs `sudo docker compose up`)

turn that off by adding yourself to the user group

my username is ubuntu so thats what ill put

```jsx
sudo usermod -aG docker ubuntu
```

sudo usermod -aG docker ubuntu breaks down to:

- usermod — modify a user account
- a — append (don't remove existing groups)
- G docker — add to the docker group
- ubuntu — the user to modify

### git cloning

copy my private key onto server vs. just make a new key

im going to go with making a new key this time.

oc40 will have its own private and public ssh key

```jsx
ssh-keygen -t ed25519 -C "oc40"
```

no passphrase for automated processes

go to github and add the public key

now everything works 😁

ssh login into oc40,

git clone like normal 👍

### test docker build

i have no `.env` , no postgres, no `host.docker.internal` 

so the container would break but i would be able to try building the image on oc40

just create the `.env` file for now at least

```jsx
touch .env
```

```jsx
cd ~/falconup26/ops
```

```jsx
docker compose -f compose.build.yaml up --build
```

it builds !!!! 🎉

clean up:

```jsx
docker compose -f compose.build.yaml down
```

---

but i think i still want to build locally and then only pull on the oracle server

because i dont want it to take forever to load

!! start with server building → switch to pull from ghcr

### install postgres on oc40

```jsx
sudo apt install postgresql -y
```

postgres installed 🎉

apt: Ubuntu's package manager

package managers !!!

| **Ubuntu** | apt |
| --- | --- |
| **MacOS** | brew |
| **Node** | npm |
| **Python** | pip / uv |

to interact with the postgres we just installed,

use the client

```jsx
sudo -u postgres psql
```

psql: postgres’s command line client

Postgres (the server) is already running in the background

sql is how you talk to it.

### set up a database and user

by default with postgres installation, `postgres` user already exists

(default superadmin)

```jsx
sudo -u postgres psql
```

opens postgres client

alter postgres user

```jsx
ALTER USER postgres WITH PASSWORD '<strong-password>';
```

create database

```jsx
CREATE DATABASE falconup;
```

quit postgres client

```jsx
\q
```

`/etc` :   Linux system-wide config settings folder

Some examples:

- /etc/postgresql/ — Postgres config
- /etc/nginx/ — nginx config
- /etc/ssh/ — SSH server config
- /etc/apt/ — package manager sources and settings
- /etc/hosts — hostname-to-IP mappings
- /etc/systemd/ — service definitions

generally things that run as background processes (daemon)

requires `sudo` to edit any of these files

!

Ubuntu is the operating system

Linux is the kernel

!

### edit postgres config

postgres host-based authentication: 

who can connect to Postgres and how they authenticate

```jsx
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

add this line to `/etc/postgresql/16/main/pg_hba.conf` 

```jsx
# TYPE  DATABASE        USER            ADDRESS                 METHOD
host    falconup        postgres        172.17.0.0/12           scram-sha-256
```

This line says: 

- allow network connections (host)

- to the falconup database,

- as user postgres,

- from IP addresses 172.17.0.0/12 (Docker's bridge network),

- using password authentication (scram-sha-256)

!! the firewall is fixed. Now it's pg_hba.conf rejecting the connection from 172.18.0.2. Same problem as the iptables — we used 172.17.0.0/16 but the container is on 172.18.x.x

now edit this file:

main settings file for postgres

```jsx
sudo nano /etc/postgresql/16/main/postgresql.conf
```

uncomment and edit this line

```jsx
#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '*'                  # what IP address(es) to listen on;
```

By default, listen_addresses = 'localhost', which means Postgres only accepts connections from the same machine on 127.0.0.1

Docker containers have their own network (the bridge at 172.17.0.x), so from Postgres's perspective they're coming from a different address, not localhost.

then restart postgres background process:

```jsx
sudo systemctl restart postgresql
```

This lets the Docker containers reach Postgres on the host
through the bridge network.

### set up .env on oc40

copy from `.env.example`

```jsx
cp ~/_proj/falconup26/.env.example ~/_proj/falconup26/.env
```

the key changes for oc40:

```jsx
PGHOST=172.17.0.1
PGUSER=postgres
PGPASSWORD=<strong-password>
PGDATABASE=falconup
```

### oc40 os firewall blocks postgres → fix

```jsx
sudo iptables -I INPUT -s 172.17.0.0/12 -j ACCEPT
```

In short: allow any traffic coming from Docker containers to reach the host.

The /12 range covers 172.16.0.0 through 172.31.255.255, which includes both 172.17.x.x (default bridge) and 172.18.x.x (your compose network) and any
future Docker networks. The previous /16 rule was too narrow.

```jsx
sudo apt install iptables-persistent -y
```

Installs a package that saves and restores firewall rules across reboots

```jsx
sudo netfilter-persistent save
```

Saves the current iptables rules to disk so they survive a reboot

This allows all traffic from Docker containers to reach the host (including Postgres). 

verify some details:

```jsx
sudo iptables -L INPUT -n --line-numbers
```

Lists all firewall rules in the INPUT chain

```jsx
docker network inspect bridge | grep Gateway
```

shows full details about Docker's default bridge network

filters the output to just show the gateway IP

Then try again:

```jsx
cd ~/_proj/falconup26/ops
docker compose -f compose.build.yaml up --build
```

### everythings fixed 🎉 docker image builds

both services running! Frontend on port 4040, backend on 40401, Alembic migrations ran successfully. The app is live on oc40.
