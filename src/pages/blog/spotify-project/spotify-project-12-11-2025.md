---
layout: ../../../layouts/BlogPost.astro
title: "Spotify Project - Day 4"
date: "2025-12-11"
description: "Continuing Flask setup + redirect URI updates"
dayNumber: 4
---

picking up where we left off

[https://www.youtube.com/watch?v=2if5xSaZJlg](https://www.youtube.com/watch?v=2if5xSaZJlg)

`main.py`

```jsx
import os

from flask import Flask

app = Flask(__name__)
app.config['SECRET_KEY'] = os.urandom(24)    # temp random key

client_id
client_secret
redirect_uri
scope

if __name__ == '__main__':
    app.run(debug=True)
```

ok now the video goes to create the spotify app thing on the developer website

but ive already done that step and have an app made

important note tho !

he sets the redirect URI to 

```jsx
http://localhost:5000/callback
```

but

“Spotify no longer supports `http://localhost` as a redirect URI for security reasons. To test locally, you must use the explicit loopback IP address `http://127.0.0.1`, followed by your port number and any desired path. ”  -gemini

so ill set it as 

```jsx
http://127.0.0.1:5000/callback
```

replacing [`localhost`](http://localhost) with `127.0.0.1`

ok now the video is saying to retrieve the client id and client secret and put them in an .env file but i did that already

only update is in .env file im going to put the new redirect URI i just changed

.env

```jsx
SPOTIFY_CLIENT_ID=
SPOTIFY_CLIENT_SECRET=
SPOTIFY_REDIRECT_URI=http://127.0.0.1:5000/callback
```