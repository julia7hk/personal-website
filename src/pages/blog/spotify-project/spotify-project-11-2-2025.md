---
title: "Spotify Project - Day 2"
date: "2025-11-02"
description: "Figuring out Spotify API authentication with spotipy and troubleshooting redirect URLs"
dayNumber: 2
---

ran

```bash
npm install next-auth axios
```

cuz idk why

i think next step is OAuth to get this spotify api thing actually working

### roadblock 1

ok im having trouble using the spotify api directly

from some research i found the `spotipy` library 

- quick break: what is spotipy??? why are we using this instead of the spotify api directly
    
    
    spotipy is a **wrapper library**
    
    The **Spotify Web API** is just a set of **HTTP endpoints**.
    
    If you used it *directly*, you’d need to:
    
    - Manually build URLs like
        
        `https://api.spotify.com/v1/me/top/tracks`
        
    - Attach headers:
        
        ```python
        headers = {"Authorization": f"Bearer {access_token}"}
        
        ```
        
    - Handle JSON parsing, pagination, and errors.
    - Refresh expired tokens.
    - Format request parameters properly.
    
    That’s **a lot of boilerplate** just to get a list of songs.
    
    vs.
    
    Spotipy does all that for you with one clean call:
    
    ```python
    sp.current_user_top_tracks(limit=10)
    
    ```
    

ok it sounds like the spotipy api will be good for me

### roadblock 2

error:

the spotify api is very picky about the redirect url

i put [localhost:3000](http://localhost:3000) but its being hosted at [localhost:3002](http://localhost:3002) or smth so thats not good

plus troubleshooting with adding [localhost](http://localhost) as a redirect url

![image.png](/blog-images/spotify-project-11-2-2025/image.png)

so im gonna follow this instead

https://youtu.be/2if5xSaZJlg?si=1fHDEzKAoeyHHdF9

navigating api documentation is a skill fr