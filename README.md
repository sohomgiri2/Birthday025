# A Little World Made for You

An interactive, offline-ready 3D birthday gift: cinematic intro, birthday cake, photo gallery (slideshow / 3D cards / memory wall / lightbox), letters, timeline, six mini-games, a 3D universe, a virtual pet, mystery gifts, a music room, a password-protected secret room, achievements, settings and a final page.

Built with **React + Vite + Tailwind** (modular TypeScript). No CDN, no external APIs — fonts and everything else are bundled. Sounds are synthesised in the browser, so no audio files are required.

> Developer: **Sohom Giri** · Instagram [@itz_sg_063](https://www.instagram.com/itz_sg_063/) · Independent Web Developer / Creative Developer

---

## 1. Project structure

```
index.html              page shell (title, manifest, icons)
public/                 copied as-is to the final site  ← put your media here
  photos/               photo1.png, photo2.png ...   (+ manifest.json)
  music/                song1.mp3, song2.mp3 ...
  letters/              letter1.png, letter2.png ...  (optional attachments)
  service-worker.js     offline cache
  manifest.json         PWA manifest      icons/  offline.html
src/
  config.ts             ★ password, letters, timeline, captions, paths
  App.tsx               navigation, swipe, menu, overlays
  sections/             one file per slide (Birthday, Gallery, Letters, Games, Universe, Pet, Gifts, MusicRoom, SecretRoom, ...)
  games/                Slice, Stars, Memory, Scratch, Wheel, TapStars  (each isolated by an error boundary)
  lib/                  datavault.ts (SG DataVault), photos.ts, music.ts, audio.ts, achievements.ts, store.ts, fx.ts, offline.ts
scripts/                generate-photo-manifest.mjs, make-icons.mjs
```

## 2. Run / build

```bash
npm install
npm run dev        # local preview
npm run build      # outputs dist/ (index.html + media + service worker)
```

## 3. Add photos (automatic)

Upload images to **`public/photos/`** named exactly:

```
photo1.png  photo2.png  photo3.png ...   (.jpg and .jpeg work too, up to 300)
```

No code edits are needed. How it works (GitHub Pages can't list folders, so two static-friendly methods are combined):

1. **Auto-detect (default):** the site probes `photo1`, `photo2`, … (png → jpg → jpeg) in small batches, skips missing files without showing broken icons, and stops after 12 consecutive missing numbers (`PHOTO_CONFIG.probeGap` in `src/config.ts`). Keep numbering continuous (or leave gaps smaller than 12).
2. **Manifest (most reliable, optional):** run `node scripts/generate-photo-manifest.mjs` to write `public/photos/manifest.json` listing every photo. If the list is not empty the site uses it directly (works with any gaps and is faster). The included GitHub Actions workflow runs this script automatically on every deploy.

The detected list is cached on the device, so the gallery opens instantly and works offline. Visitors can also add pictures from their own device (stored in IndexedDB).
Change folder / names / limits in `PATHS` and `PHOTO_CONFIG` (`src/config.ts`). Captions: `CAPTIONS` in `src/config.ts` (e.g. `1: "The day we got lost"`).

## 4. Add music

Put `song1.mp3`, `song2.mp3`, … (also `.ogg` / `.m4a`) in **`public/music/`**. They appear in the Music Room playlist automatically. Pretty titles: `MUSIC_CONFIG.titles` in `src/config.ts`. Nothing ever autoplays before the first tap. If no files exist, a built-in ambient track is offered instead and a friendly empty state is shown.

## 5. Letters

Edit `LETTERS` in `src/config.ts` (title, text, optional action button). Attach an image to a letter by either:
- uploading `public/letters/letter1.png` (number = position in the list; png / jpg / jpeg), or
- setting `photo: 3` (uses `photo3`) or `image: "letters/my-pic.png"` on that letter.

"A Small Favor to Ask" (the letter to Gayatri) is the 2nd letter, with the "Okay, I'll Think About It" button. It only says thanks — no pressure.

## 6. Secret-room password

Set it in **one place**: `SECRET.password` in `src/config.ts` (case-insensitive; default `smile`). Also edit `SECRET.hint` and `SECRET.letter`.
⚠️ A client-side password is only a fun privacy barrier — anyone who reads the page's source can find it. Never use it to protect real secrets.

## 7. Offline use

- A service worker (`public/service-worker.js`) caches the page, the app, icons, photos, letters and music (cache-first / stale-while-revalidate) and shows `offline.html` if the app was never cached.
- Open the site once while online (needs HTTPS — GitHub Pages is). After that it works offline. In **Settings → Offline mode → "Cache photos & music"** forces everything to be saved.
- It is installable (PWA) on Android/Chrome and via "Add to Home Screen" on iOS.
- After you upload new photos, a visitor needs one online visit to receive them. Bump `VERSION` in `service-worker.js` if you want every device to refresh its whole cache.
- Limit: a website can't discover brand-new files offline — the photo list is cached/bundled from the last online visit.

## 8. Deploy with GitHub Pages

**Option A — automatic (recommended):**
1. Create a GitHub repository and upload *all* project files (including `.github/`).
2. Repo → **Settings → Pages → Source: GitHub Actions**.
3. Push to `main`. The workflow builds the site and publishes it at `https://<user>.github.io/<repo>/`.

**Option B — manual:** run `npm run build`, then upload the *contents of `dist/`* to a branch (e.g. `gh-pages`) and select it in Settings → Pages.

All internal paths are relative, so it works in a repo sub-folder too.

## 9. Storage — SG DataVault

`src/lib/datavault.ts`: localStorage for settings/progress, IndexedDB for added photos and larger records, with automatic saving, JSON backup/restore, storage usage, and confirmed reset/clear (Settings page). Storage failures are handled gracefully (falls back to memory).

## 10. Notes & limitations

- The 3D parts (gift box, envelope, orb, photo ring, universe) use CSS 3D transforms and Canvas 2D instead of WebGL: lighter on battery and works everywhere, so no WebGL fallback is needed.
- Sound effects are generated live with the Web Audio API (works offline, nothing to upload). Browsers only start audio after a tap.
- Surprise popups are capped per session and never appear during games; frequency can be changed or switched off in Settings.
