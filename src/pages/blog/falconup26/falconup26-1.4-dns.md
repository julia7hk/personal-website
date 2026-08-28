---
title: "FalconUp 2026 - dns"
date: "2026-07-02"
description: "Nginx as a reverse proxy in the container, Cloudflare DNS, and getting falconup.julia7hk.com to actually resolve"
dayNumber: 4
milestone: 1
milestoneTitle: "project foundation"
---

# DNS

### domain wiring + TLS

TLS: Transport Layer Security

the secure HTTP

**‼️ cloudflare handles TLS for you**

but what is reverse proxy anyway?

**Reverse Proxy:**  

sits in front of all your service ports (frontend port, backend port)

routes internet traffic to appropriate port based on rules

e.g. Caddy

e.g. **Nginx**

- users visit [`falconup.julia7hk.com`](http://falconup.julia7hk.com) instead of `falconup.julia7hk.com:4040`
    
    (no port number)
    

[website domains (DNS) 6/22/2026](https://app.notion.com/p/website-domains-DNS-6-22-2026-387812596e258088a4c8e2942378fc1f?pvs=21)

DNS: Domain Name System

human-readable domain names ([google.com](http://google.com/))  ←→  IP addresses

DNS A Records:  specific DNS entry that does the mapping

| TYPE | NAME | CONTENT |
| --- | --- | --- |
| A | falconup | << nc01’s public IP >> |

!!! you can find the public ip address by

- ssh into the server
- run `curl -4 [ifconfig.me](http://ifconfig.me/)`

DNS CNAME Records:  maps domain name ←→ another domain name

| TYPE | NAME | CONTENT |
| --- | --- | --- |
| CNAME | falconup | falconup.netlify.app |

### pr steps:

set up nginx

- set up nginx within the docker container (don’t install nginx on oc40)
- frontend port, backend port
- copy structure from kkulgag project’s nginx configuration

update port numbers to work on oc40 server !!

### oracle cloud config to port 80 for nginx

go to oracle cloud

go to instance oc40

find security list

Default Security List for vcn-20220911-2137

go to security rules

go to ingress rules

add ingress rules

### configure oc40 after nginx merged to main

pull the nginx commit merged to main

```jsx
 git pull
```

```jsx
sudo iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```

```jsx
sudo netfilter-persistent save
```

build image on oc40

```jsx
docker compose -f compose.build.yaml up --build -d
```

then check [`http://falconup.julia7hk.com`](http://falconup.julia7hk.com/)

![Site not resolving yet](/blog-images/falconup26-1.4-dns/image.png)

somethings wrong T^T

‼️ cloudflare was set to nc01’s public ip, not oc40’s ip 💀

omg it works now tho 😁

![falconup.julia7hk.com loading after the Cloudflare IP fix](/blog-images/falconup26-1.4-dns/image%201.png)
