---
layout: ../../layouts/BlogPost.astro
title: "SASE Sniping Discord Bot - Day 2"
date: "2025-10-23"
description: "Setting up the Python environment and building the basic bot structure with scheduling functionality"
dayNumber: 2
---

now that i have a discord bot made, i need the actual bot logic for it to run

---

## step 1. setup python environment

`cd` into the correct directory. then run:

```bash
python3 -m venv venv
```

```bash
source venv/bin/activate
```

^ setting up python environment

```bash
pip install discord.py python-dotenv schedule
```

^ installs 3 libraries

## step 2. .env file

```bash
DISCORD_TOKEN=your_bot_token_here
```

^ run direnv allow 


## step 3. basic bot setup

```python
import os
import discord
from discord.ext import tasks, commands
from dotenv import load_dotenv
import datetime

load_dotenv()
TOKEN = os.getenv("DISCORD_TOKEN")

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)
```

## step 4. schedule data structure

in `bot.py`:

```python
# Message template
MESSAGE_TEMPLATE = "{name} has class right now in {location}! If you're nearby, go say and snipe them!"

# 24-hour HH:MM format, local server time
class_schedule = {
    "Group 1": {
        "Alice": [
            ("Mon", "13:30", "14:20", {"location": "WTHR 172"}),
            ("Wed", "13:30", "14:20", {"location": "WTHR 172"}),
            ("Fri", "13:30", "14:20", {"location": "WTHR 172"}),
        ],
        "Alan": []
    },

    "Group 2": {
        "Bob": []
    },

    "Group 4": {
        "Claire": [
            ("Tue", "11:30", "14:20", {"location": "the 4th floor of CHAS"}),
            ("Thu", "11:30", "14:20", {"location": "the 4th floor of CHAS"}),
        ]
    }
}

```