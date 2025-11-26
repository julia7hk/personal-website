---
layout: ../../layouts/BlogPost.astro
title: "SASE Sniping Discord Bot - Day 6"
date: "2025-11-10"
description: "Fixing duplicate messages, adding start/end notifications, and resolving timezone issues on Oracle Cloud server"
dayNumber: 6
---

ok the bot is running on the oracle server from just running the [`bot.py`](http://bot.py) script 24/7 but we have some issues to fix

## 1.

![image.png](/blog-images/sase-sniping-discord-bot-day-6/image.png)

the bot sends a message every minute

### fix!

first, a start and end message:

```python
# Message templates
MESSAGE_TEMPLATE_START = "{name} has class starting in 10 minutes at {location}! If you're nearby, go say hi and snipe them!"
MESSAGE_TEMPLATE_END = "{name}'s class ends in 10 minutes at {location}! If you're nearby, go say hi and snipe them!"
```

rewrite logic to check if current time == 10 mins before start or end

now `git pull` in the oracle server

back up and running…

hope that helps

## 2.

![image.png](/blog-images/sase-sniping-discord-bot-day-6/image%201.png)

sent out at 2:30 am??? 

double checked the class schedule data:

```python
"Group 4": {
        "Parin": [
            ("Mon", "07:30", "08:20", {"location": "BHEE 224"}),
            ("Wed", "07:30", "08:20", {"location": "BHEE 224"}),
            ("Fri", "07:30", "08:20", {"location": "BHEE 224"}),
            .
            .
            .
```

so it’s properly entered i didnt make a mistake

potential issue: the code is based on the local computer’s time and timezone right now

! vthe oracle cloud server oc03 is based in korea right now

let’s try fixing that…

```python
from zoneinfo import ZoneInfo
```

```python
# Use Eastern Time
eastern = ZoneInfo("America/New_York")
now = datetime.datetime.now(eastern)
```


---

it’s working like this so far 😋

![image.png](/blog-images/sase-sniping-discord-bot-day-6/image%202.png)