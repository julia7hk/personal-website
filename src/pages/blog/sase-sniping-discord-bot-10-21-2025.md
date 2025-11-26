---
layout: ../../layouts/BlogPost.astro
title: "SASE Sniping Discord Bot"
date: "2025-10-21"
description: "Building a Discord bot to automatically notify mentors when their class time arrives"
dayNumber: 1
---

group 3 has discovered a few classes for each of the mentors

currently nothing is really being done with them and they just sit as buried discord messages

i had an idea to make a discord bot that automatically sends a message to say it was time for someone's class

---

first go to discord and make an application for a discord app
https://discord.com/developers/applications/

### selected app

obtain
- application ID
- public key


### bot tab

obtain
- bot reset token


### OAuth2 tab

choose permissions for the bot to have

permissions chosen

Scopes: 
- bot

Bot permissions:
- send messages

Integration type: guild install (discord bot in a server)

**obtain: generated url** 

enter url into browser, add it to a server

![Discord bot added to server](/blog-images/sase-sniping-discord-bot/image.png)


yayy made a bot