# US Open Weekend — installable itinerary

Static PWA. No build step, no dependencies. Push it as-is.

```
index.html      the whole app
manifest.json   name, colors, icons
sw.js           offline cache
icons/          192 / 512 / maskable / apple-touch
```

## Deploy to GitHub Pages

```bash
cd itin
git init
git add .
git commit -m "US Open weekend itinerary"
git branch -M main
git remote add origin https://github.com/19awburris88/itinerary.git
git push -u origin main
```

Then in the repo: **Settings → Pages → Source: Deploy from a branch → main / (root)**.
Live at `https://19awburris88.github.io/itinerary/` in a minute or two.

Any static host works — Netlify, Vercel, Cloudflare Pages. It just has to be HTTPS;
Chrome won't offer to install over plain HTTP.

## Install on Android

1. Open the URL in Chrome.
2. Menu (⋮) → **Install app** — or **Add to Home screen** on older builds.
3. Launch it from the home screen. No address bar, navy status bar, own entry in the app switcher.

If **Install app** doesn't appear: confirm you're on HTTPS, hard-refresh once, and check
DevTools → Application → Manifest for a warning.

Adrienne can install it from the same URL on her phone. The checklist is per-device —
her notes won't sync to yours.

## Offline

The service worker caches the page and fonts on first load, so it opens on the 7 train
and inside the grounds where service drops. Load it once on wifi before you fly.

## After you edit index.html

Bump the cache name in `sw.js` (`usopen-v1` → `usopen-v2`) and push. Without that, phones
that already installed it may keep serving the old copy.

## Notes

- Checklist and planning notes save to `localStorage` on the device.
- Dates are hardcoded to September 2026 — the countdown and the auto-selected day key off it.
- No analytics, no network calls beyond Google Fonts.
