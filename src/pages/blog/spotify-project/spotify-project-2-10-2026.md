---
layout: ../../../layouts/BlogPost.astro
title: "Spotify Project - Day 8"
date: "2026-02-10"
description: "Fixing redirect URI issues and new UI work"
dayNumber: 8
---

for the longest time i had an issue where i couldnt get past the login screen when i ran the app or i would run into a `INVALID_CLIENT: Invalid redirect URI` error.

the issue is clearly with the redirect URI 

but the problem was that i had used `localhost` throughout my codebase and then in the spotify developer settings for the app i had used **`127.0.0.1`** because using `localhost` was deprecated after a certain point

but your codebase and the set redirect URIs have to be an exact match.

so update your codebase to also use **`127.0.0.1` .**

you also need to visit `http://127.0.0.1:3000` instead of using `http://localhost:3000`.

### update `localhost` —> `127.0.0.1`

<a href="https://github.com/julia7hk/spotify/commit/c63cced06bf732bf3e2c3b9e01ed28e4dea728cb" target="_blank">https://github.com/julia7hk/spotify/commit/main</a>

### steps to run:

1. flask server

```jsx
(venv) ➜  spotify git:(main) ✗ python3 main.py
 * Serving Flask app 'main'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 *** Running on http://127.0.0.1:5001**
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 192-494-816

```

1. next.js

```jsx
➜  spotify git:(main) ✗ npm run dev

> spotify@0.1.0 dev
> next dev

   ▲ Next.js 15.3.2
   - Local:        http://localhost:3000
   - Network:      http://10.186.174.37:3000
   - Environments: .env

 ✓ Starting...
 ✓ Ready in 1079ms

```

**! but go to `http://127.0.0.1:3000` instead**

![Login screen working](/blog-images/spotify-project-2-10-2026/image.png)

yay i can see the ui now!!

---

# new ui work

![New UI work](/blog-images/spotify-project-2-10-2026/image%201.png)

![New UI work continued](/blog-images/spotify-project-2-10-2026/image%202.png)