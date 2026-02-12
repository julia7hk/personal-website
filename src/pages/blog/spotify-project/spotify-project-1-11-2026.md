---
title: "Spotify Project - Day 6"
date: "2026-01-11"
description: "Connecting Flask API to Next.js frontend and fixing port issues"
dayNumber: 6
---

i dont know what day to call this anymore so im just gonna call it the 6th one

---
can you believe im on a plane right now

working on a personal project

thats crazy

im heading back to campus after winter break

its time for spring semester

---

> i want to connect the working get-playlists flask call to the next app
> 

blank next app

![Blank Next.js app](/blog-images/spotify-project-1-11-2026/image.png)

add flask cors to requirments.txt

```jsx
Flask==3.0.2
flask-cors==6.0.2
spotipy==2.23.0
```

this also means go into venv and pip install flask-cors

here’s the [main.py](http://main.py) changes that were made to create a new flask api endpoint for next

```jsx
@app.route('/api/playlists')
def api_playlists():
    if not sp_oauth.validate_token(cache_handler.get_cached_token()):
        return jsonify({'authenticated': False}), 401

    playlists = sp.current_user_playlists()
    user = sp.current_user()

    return jsonify({
        'authenticated': True,
        'user': {
            'name': user['display_name'],
            'image': user['images'][0]['url'] if user['images'] else None
        },
        'playlists': [{
            'id': pl['id'],
            'name': pl['name'],
            'url': pl['external_urls']['spotify'],
            'image': pl['images'][0]['url'] if pl['images'] else None,
            'tracks': pl['tracks']['total']
        } for pl in playlists['items']]
    })

@app.route('/api/auth-url')
def api_auth_url():
    return jsonify({'url': sp_oauth.get_authorize_url()})

```

as of this point, no changes on next.js side, so nothing will be visible from there

create a simple react ui

to get it to run, you need to have both the flask api and next app running

```jsx
➜  spotify git:(nextjs-ui) ✗ source venv/bin/activate
(venv) ➜  spotify git:(nextjs-ui) ✗ 
(venv) ➜  spotify git:(nextjs-ui) ✗ python3 main.py
```

```jsx
➜  spotify git:(nextjs-ui) ✗ npm run dev 
```

![UI is there](/blog-images/spotify-project-1-11-2026/image%201.png)

ui is there !

but when i clicked the button i got an error saying failed to fetch 😔

![Failed to fetch error](/blog-images/spotify-project-1-11-2026/image%202.png)

it turns out it was something about port 5000 is already occupied by some OS program

so change all the port 5000 flask api things to port 5001 instead

change:

- .env
- spotify developer page app settings
- main.py
    
    ```jsx
    if __name__ == '__main__':
        app.run(debug=True, port=5001)
    ```
    
- and maybe other things

but now it works !

![Spotify auth working](/blog-images/spotify-project-1-11-2026/image%203.png)

after approving it redirects to the page we had before from fetching the results

![Playlists displayed](/blog-images/spotify-project-1-11-2026/image%204.png)

i think this is progress 🤞