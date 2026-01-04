---
layout: ../../../layouts/BlogPost.astro
title: "Spotify Project - Day 3"
date: "2025-11-27"
description: "Setting up Flask API with spotipy for Spotify authentication using authorization code flow"
dayNumber: 3
---

today is thanksgiving technically !

im just glad to be on break tbh

---

where we left off was i was having trouble with the api so i was going to follow this youtube tutorial

https://youtu.be/2if5xSaZJlg?si=1fHDEzKAoeyHHdF9

let’s see how far along i get today

---

- this will be a flask api + spotipy combo

- 2 different ways of handling authentication (oauth flows) with spotipy api
    - authorization code
    - client credentials
- client credentials seems somewhat limiting, so im going to go with **authorization code**

---

i did some research on what other spotify listening summarizing apps there already are

its listed in the central spotify project notion page

thats where i spent some of my time instead of following the tutorial

---

started coding

## initial setup

1. create new directory and [`main.py`](http://main.py) file

2. creating a python3 virtual environment

```jsx
python3 -m venv venv
```

^ same command ; used it for the sase discord bot project too

this creates a `/venv` directory

3. create `requirments.txt` file

```jsx
Flask==3.0.2
spotipy==2.23.0
```

4. install dependencies

activate virtual environment

```jsx
source venv/bin/activate
```

^ same command ; used it for the [sase discord bot project](/blog/sase-sniping-discord-bot-day-2-10-23-2025) too

now terminal looks like

```jsx
(venv) ➜  spotify git:(main) ✗
```

`venv` is the name of the virtual environment you’re in right now

now `pip install`  the dependencies within the virtual environment

```jsx
pip install -r requirements.txt
```



## actual logic

1. flask app setup

```jsx
import os

from flask import Flask

app = Flask(__name__)

if __name__ == '__main__':
    app.run(debug=True)
```

quick check with

```jsx
python3 main.py
```

it’s running ! but theres nothing there yet

![Flask app running](/blog-images/spotify-project-11-27-2025/image.png)

2. set up flask session

but maybe next time, its time to sleep 😴