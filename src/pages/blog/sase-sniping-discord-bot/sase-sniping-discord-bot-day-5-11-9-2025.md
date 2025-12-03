---
layout: ../../../layouts/BlogPost.astro
title: "SASE Sniping Discord Bot - Day 5"
date: "2025-11-09"
description: "Running the bot with pyenv and screen, updating class schedule data, and testing notifications"
dayNumber: 5
---

server day 2

contining on from installing pyenv

```bash
ubuntu@oc03:~$ pyenv versions
* system (set by /home/ubuntu/.pyenv/version)
  3.11.14
ubuntu@oc03:~$ pyenv global 3.11
ubuntu@oc03:~$ pyenv versions
  system
* 3.11.14 (set by /home/ubuntu/.pyenv/version)
ubuntu@oc03:~$ pip

Usage:
  pip <command> [options]
```

cd into proj directory

```bash
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$ python bot.py
2025-11-09 16:54:35 INFO     discord.client logging in using static token
2025-11-09 16:54:37 INFO     discord.gateway Shard ID None has connected to Gateway (Session ID: af05165e1aa37168a342bf9725adf18f).
Logged in as sase-sniping#4376

```

use screen

```bash
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$ screen
[detached from 366536.pts-0.oc03]
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$
```

```bash
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$
ubuntu@oc03:~/_proj/sase-sniping-discord-bot$ python bot.py
2025-11-09 16:55:21 INFO     discord.client logging in using static token
2025-11-09 16:55:23 INFO     discord.gateway Shard ID None has connected to Gateway (Session ID: d01e8980889de18189decba6e911a830).
Logged in as sase-sniping#4376

```

[`bot.py`](http://bot.py) is still running throughout the entire day 😁

![Bot running in screen session](/blog-images/sase-sniping-discord-bot-day-5/image.png)

let’s hope this actually works and sends out messages to me 😭 ill be looking out for messages throughout the day

---

inputted actual class schedule data of all the mentors that are in the group3 server so far

log back in to oc03 server

```bash
ssh ubuntu@oc03
```

get into the screen that was running `bot.py`

```bash
screen -rx
```

pull the github repo changes

```bash
git fetch origin
git pull
```

! reminder to set up aliases on the server (later --> [blog: set up aliases on the server](/blog/alias-11-20-2025))

now it should have updated class schedule data

we’ll see if this works 🤞
