---
layout: ../../../layouts/BlogPost.astro
title: "Spotify Project - Day 7"
date: "2026-01-13"
description: "Expanding API endpoints for top artists, tracks, recently played, and listening profile"
dayNumber: 7
---

today i feel like expanding the api calls instead of working on the frontend

here are some steps i took

---

## 1. fix scopes

BEFORE

- get public playlists

```jsx
scope = 'playlist-read-private'
```

AFTER

- get public playlists
- get top artists/tracks
- get recently played

```jsx
scope = 'playlist-read-private user-top-read user-read-recently-played'
```

## 2. new api endpoints

@app.route('/api/top-artists')

![Top artists API response](/blog-images/spotify-project-1-13-2026/image.png)

@app.route('/api/top-tracks')

![Top tracks API response](/blog-images/spotify-project-1-13-2026/image%201.png)

@app.route('/api/recently-played')

![Recently played API response](/blog-images/spotify-project-1-13-2026/image%202.png)

wow i literally finished listening to that song like 2 seconds ago !

## 3. add a user profile api endpoint

@app.route('/api/me')

![User profile API response](/blog-images/spotify-project-1-13-2026/image%203.png)

for dashboard ui purposes later i suppose

## 4. listening profile

there used to be a `/audio-features` api endpoint but it got deprecated

```jsx
@app.route('/api/listening-profile')
def api_listening_profile():
    if not sp_oauth.validate_token(cache_handler.get_cached_token()):
        return jsonify({'authenticated': False}), 401

    top_tracks = sp.current_user_top_tracks(limit=20)
    top_artists = sp.current_user_top_artists(limit=20)

    # ---- Genre aggregation ----
    genre_count = {}
    for artist in top_artists['items']:
        for genre in artist['genres']:
            genre_count[genre] = genre_count.get(genre, 0) + 1

    # ---- Track metadata ----
    tracks = []
    for t in top_tracks['items']:
        tracks.append({
            'name': t['name'],
            'popularity': t['popularity'],
            'duration_ms': t['duration_ms'],
            'explicit': t['explicit'],
            'release_year': t['album']['release_date'][:4]
        })

    avg_popularity = sum(t['popularity'] for t in tracks) / len(tracks)
    avg_duration = sum(t['duration_ms'] for t in tracks) / len(tracks)

    return jsonify({
        'top_genres': sorted(
            genre_count.items(),
            key=lambda x: x[1],
            reverse=True
        )[:10],
        'avg_popularity': round(avg_popularity, 1),
        'avg_duration_min': round(avg_duration / 60000, 2),
        'explicit_ratio': round(
            sum(1 for t in tracks if t['explicit']) / len(tracks),
            2
        ),
        'release_years': [t['release_year'] for t in tracks]
    })

```

![Listening profile API response](/blog-images/spotify-project-1-13-2026/image%204.png)