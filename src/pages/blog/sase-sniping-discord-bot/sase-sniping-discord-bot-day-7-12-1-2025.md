---
layout: ../../../layouts/BlogPost.astro
title: "SASE Sniping Discord Bot - Day 7"
date: "2025-12-01"
description: "Fixing Discord bot permissions by using channel ID instead of channel name"
dayNumber: 7
---

we're finally back from thanksgiving break

new snipe groups released

but the bot didn’t send a message in the group3 server and only sent one in my personal server

what’s the problem?

probably discord server perms

i went into server settings, channel settings, role settings, member settings

did literally everything i could to allow this bot to have perms to access channels and send messages

but it still didnt work

whats the problem???

.

.

.

instead of find channel by channel name, find channel by channel id instead

before:

```jsx
channel = discord.utils.get(bot.get_all_channels(), name="snipe-bot")
```

after:

```jsx
channel = bot.get_channel(1439880595424415765)
```

---

to find a discord channel’s channel id:

1. turn on developer mode on your discord account
    1. go to user settings
    2. search developer
2. right click on the channel you want to find the id of
3. copy channel id

and it works :DD

![image.png](/blog-images/sase-sniping-discord-bot-day-7/image.png)