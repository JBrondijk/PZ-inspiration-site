# Random moment — YouTube playlist

A tiny static page that picks a **random video** from a fixed YouTube playlist
and loads it **paused at a random timestamp** taken from the video's
description, skipping any timestamp labelled *intro* or *outro*. A **Shuffle**
button rerolls (new random video + timestamp) without reloading.

The playlist is hard-coded in `app.js` (`PLAYLIST_ID`).

## How it works

The YouTube embed / IFrame Player API can't read descriptions or chapters, so
the page calls the **YouTube Data API v3** directly from the browser:

- `playlistItems.list` → the playlist's video ids
- `videos.list` (`part=snippet`) → each video's `description`; timestamps are
  then regex-parsed from the text and intro/outro entries are dropped.

A video is only eligible if it has at least one non-intro/outro timestamp. The
built dataset is cached in `localStorage` for 6 hours, so loads and shuffles
don't re-hit the API. Add `?refresh=1` to the URL to force a rebuild.

## Setup

1. In the [Google Cloud Console](https://console.cloud.google.com/), create a
   project and **enable the YouTube Data API v3**.
2. Create an **API key** and **restrict it**:
   - **Application restriction:** HTTP referrers — allow your serving origin,
     e.g. `http://localhost:8000/*`.
   - **API restriction:** YouTube Data API v3 only.
3. Add the key locally (the file is gitignored):
   ```sh
   cp config.example.js config.js
   # then edit config.js and paste your key
   ```

## Run

```sh
python3 -m http.server 8000
```

Open <http://localhost:8000/>. Without a valid key the page shows a setup
message — that's expected until you finish the steps above.

## Notes

- **Quota:** one dataset build costs about `2 × ceil(videos / 50)` units, far
  under the default 10,000/day, and is cached for 6 hours.
- **Key safety:** `config.js` holds your key and is gitignored — never commit
  it. The key does reach the browser, but the HTTP-referrer restriction keeps it
  unusable from any other origin.
- `yt-dlp` is intentionally not used; everything runs live against the Data API.
