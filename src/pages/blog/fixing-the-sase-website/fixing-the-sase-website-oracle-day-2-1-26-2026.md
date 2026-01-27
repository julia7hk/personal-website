---
layout: ../../../layouts/BlogPost.astro
title: "Fixing the SASE Website - Oracle Day 2"
date: "2026-01-26"
description: "Continuing the Oracle Cloud VM setup - configuring storage settings and boot volume options"
dayNumber: 8
---

hello its been a month since i am revisiting this

where we left off: we were creating the VM instance with the free oracle cloud account but then i didnt know what ssh key to tie to the account

but i think what ive decided is i can put multiple ssh keys as authorized keys so ill put my own key and ill also put a new key and i can also add anyone else’s key if they want (if they have)

so back to creating the instance !

refer to [oracle day 1](/blog/fixing-the-sase-website/fixing-the-sase-website-oracle-day-1-12-27-2025) for exact details for the following:

1. basic information
2. security
    1. **Shape: VM.Standard.A1.Flex** (Always Free-eligible)
    2. **Shape build: Virtual machine, 2 core OCPU, 12 GB memory, 2 Gbps network bandwidth**
3. networking
    1. **Advanced options**
        1. **DNS record: Do not assign a private DNS record**
        2. **Launch options: Let Oracle Cloud Infrastructure choose the best networking type**

## 4. storage

leave default settings

### Boot volume

(”a detachable device that contains the image used to boot the compute instance.”)

❌ **Do NOT** check “Specify a custom boot volume size”

- Default **~46.6 GB** is **plenty**

✅ **Use in-transit encryption**

- Leave **ON** (default)
- Free and automatic

❌ **Do NOT** choose “Encrypt this volume with a key that you manage”

- Oracle-managed keys are fine

### Block Volumes

- ❌ Do NOT attach a block volume

### ! note: **Public IPv4 address should be set to** ✅ **YES but it cant right now**

Just make sure you **do not forget** to do these later:

- Assign a public IPv4
- Open ports 80 / 443 in the security list

press create !!

all the Ampere A1 availability domains are out of capacity rn :(

i have to try again later