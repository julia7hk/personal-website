---
layout: ../../../layouts/BlogPost.astro
title: "Fixing the SASE Website - Oracle Day 1"
date: "2025-12-27"
description: "Exploring Oracle Cloud Free Tier as an alternative to HostGator for hosting the SASE Purdue WordPress site"
dayNumber: 7
---

12/26/2025

ok so its actually the same day as the hostgator support chatting day but later time

and im still hung up on “i hate hostgator $18/mo” and “i love free oracle cloud”

so i was looking into how much the free tier of oracle cloud gives you

and how big of a server you actually need for a wordpress site

## Typical Server Sizes for WordPress

### Small / Personal Site or Blog

**Traffic:** up to ~5,000 visits/month

**Minimum server:**

- **CPU:** 1 vCPU
- **RAM:** 1 GB (2 GB recommended)
- **Storage:** 10–20 GB SSD
- **Bandwidth:** ~1 TB/month
- **Cost:** ~$5–10/month (shared or basic VPS)

✅ Works fine with:

- Lightweight theme
- Few plugins
- Caching plugin enabled

## Typical Oracle Cloud Free Tier

“

Yes, it’s actually free. If you stick within the Always Free tier limits, you won’t be charged. 

This means you can run a 4-core, 24GB machine with a 200GB disk 24/7 and it should not cost you anything. 

Or you can split those limits into 2 or 4 machines if you want.

“

so lets try it???

---

erm it timed out and i couldnt do anything

new day!

12/27/2025

## 1. create free oracle cloud account w/ sase

inputted my actual address and card information for now,,

account created !

now log in with *email* and password

yippee ! logged in

![Oracle Cloud dashboard](/blog-images/fixing-the-sase-website-oracle-day-1/image.png)

## 2. create new instance

go to instances

(home —> compute —> instances) 

((or lowk just search for instances))

click `create instance`

## 1️⃣ Basic Information

**Name**

- ✅ `wordpress` (perfect)

**Compartment**

- ✅ `sasepurdue (root)`
    
    Keep it simple unless you’re managing many resources.
    

---

## 2️⃣ Placement

**Availability Domain**

- ✅ **Any AD (AD 1 / AD 2 / AD 3)**
    
    Does *not* matter for WordPress.
    

👉 Pick **AD 1** and move on.

---

## 3️⃣ Capacity Type

- ✅ **On-demand capacity**

❌ Do **NOT** use:

- Preemptible (can be killed randomly)
- Dedicated host (paid)
- Capacity reservation (unnecessary)

---

## 4️⃣ Image and Shape

selected

Image: **Canonical Ubuntu 22.04 Minimal aarch64**

Shape: **VM.Standard.A1.Flex**

---

enable protection cuz why not 

---

**✅ Step 3: Networking — What to Choose**

**VNIC name:** leave blank (Oracle auto-names it)

**✅ Create new virtual cloud network**

**✅ Create new public subnet**

---

**✅ Step 3.5: SSH Keys**

.

.

.

im concerned that teaching the next person about ssh keys if they know absolutely nothing might be too complicated…

is this entire setup too complicated for a non technical person?

i think its still worth a try 😊

to be continued !