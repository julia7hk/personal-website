---
layout: ../../../layouts/BlogPost.astro
title: "Spotify Project - Day 1"
date: "2025-11-01"
description: "Starting up my Spotify dashboard project to track listening stats and get better recommendations"
dayNumber: 1
---

it was halloween

i got dropped off at my dorm

but i feel like i can do work rn

so but like here i am

i started this project over the summer

the original idea was i wanted to know my listening stats and also get better recommendations on new songs

so i wanted to create my own spotify dashboard

but i dont yet know what is within the realms of possibility with the spotify api

so here we are just experimenting with whats possible

but i didnt get to do much meaningful work with it then

so ill try starting it back up now !

i already have a next.js project set up

a spotify app has already been created. i have a 

- `CLIENT_ID` 
- `CLIENT_SECRET` 

### actual new stuff now ! yay

set up environment variables

```bash
SPOTIFY_CLIENT_ID=your_client_id_here
SPOTIFY_CLIENT_SECRET=your_client_secret_here
SPOTIFY_REDIRECT_URI=http://localhost:3000/api/callback
```

what’s a redirect url for…?

When you click “Connect with Spotify,” you’re actually being redirected to `SPOTIFY_REDIRECT_URI`