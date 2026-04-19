# Track Collector+ for Yandex Music

A browser console script to export tracks from Yandex Music web playlists to TXT/CSV files.

Main use case: export your "Liked" playlist (or any other playlist) into a portable format and use it for backup, analysis or migration to another service.

> ⚠️ The script does not use any private APIs. It only reads the HTML that is already rendered in your browser.

---

## Features

- Export tracks from any Yandex Music web playlist (including "Liked").
- Handles large playlists (hundreds of tracks).
- Collects:
  - track title;
  - artists;
  - track ID (when available).
- Works around "weird" rows where:
  - there is no direct /track/... link;
  - track data is hidden in aria-label and alt attributes.
- Exports to:
  - tracks-full.txt — lines in Artist – Title format;
  - tracks-full.csv — columns Artist, Title, Source.

---

## How to use

1. Open a Yandex Music playlist

   In your browser, open the Yandex Music web UI and navigate to the playlist you want to export (for example, "Liked").

2. Open DevTools → Console

   - Chrome: F12 or Ctrl+Shift+I (Cmd+Option+I on Mac), then go to the Console tab.

3. Allow pasting

   In the console, run:

   ```js
   allow pasting
   ```

   This is needed because Chrome warns you against blindly pasting code. Here we explicitly confirm we know what we are doing.

4. Paste the script

   - Open [ym-track-collector-plus.js](./ym-track-collector-plus.js) from this repo.
   - Copy its entire content.
   - Paste it into the console and press Enter.

5. Scroll through the playlist

   A small Track Collector+ panel should appear in the top-right corner of the page.

   Slowly scroll down through the playlist to the very bottom so that Yandex Music loads all tracks.

   The counter in the panel will increase as tracks are being collected.

6. Finalize capture

   When you reached the bottom (and optionally scroll a bit up and down to be safe):

   - Click Finalize capture in the panel.
   - Wait until the status changes to Ready to download.

7. Download files

   Once finished:

   - Click Download .txt to save tracks-full.txt (good for quick reading/copying).
   - Click Download .csv to save tracks-full.csv (good for spreadsheets/scripts).

---

## What the script does under the hood

1. Injects a floating panel #ym-scraper into the page with a counter and buttons.
2. Locates the playlist scroll container (virtualized list).
3. On scroll (with debounce), calls extractTracks().
4. extractTracks() iterates over track rows (.CommonTrack_root__i6shE) and for each row:
   - tries to read:
     - /album/.../track/... link → trackId;
     - title from span.Meta_title__GGBnH;
     - version/remix from span.Meta_version__c2sHU;
     - artists from span.Meta_artistCaption__JESZi inside div.Meta_artists__VnR52;
   - if data is missing, switches to fallback mode:
     - reads title from img[alt^="Трек ..."];
     - extracts artists from aria-label by subtracting the title;
     - generates a synthetic ID (fallback::...) to avoid duplicates.
5. Stores all tracks in a Map to prevent duplicates.
6. On Download .txt / Download .csv, builds the file content and triggers a download via Blob.

---

## Limitations

- The script relies on the current DOM structure of Yandex Music web:
  - classes like .CommonTrack_root__..., .Meta_title__..., .Meta_artistCaption__...;
  - attributes such as aria-label, alt, etc.
- If Yandex significantly changes its frontend markup, selectors will need to be updated.
- In tests on a large "Liked" playlist:
  - the script reliably collects ~98% of tracks;
  - a small number of tracks may still be missed (special cards, episodes, non-standard rows).
- The script runs entirely in your browser:
  - no external network calls;
  - no data is sent anywhere.

---

## Credits

Inspired by the original project idaniil24/ym-track-collector, but heavily adapted for the current Yandex Music DOM and extended with fallback parsing logic for non-standard playlist rows.

---

## Disclaimer
This is an unofficial tool.  
Use it at your own risk, and only for your own personal playlists.

Licensed under the MIT License — see LICENSE file for details.
