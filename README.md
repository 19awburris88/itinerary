# Atlanta Cigar Week — installable itinerary

Static PWA. No build step, no dependencies. Push it as-is.

```
index.html      the whole app
manifest.json   name, colors, icons
sw.js           offline cache
icons/          192 / 512 / maskable / apple-touch
fonts/          5 self-hosted woff2, latin subset
```

## Deploy to GitHub Pages

```bash
cd itin
git init
git add .
git commit -m "Atlanta Cigar Week itinerary"
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

Anyone you're travelling with can install it from the same URL.

## Sharing the checklist

**Share my notes** packs your checklist into a link and hands it to the share sheet (or the
clipboard). Text it over; the other phone opens it and gets a **Shared plan** panel listing
only what it doesn't already have, with Merge and Ignore.

No server, no account, no token — the whole payload rides in the URL fragment, which never
leaves the device until you send it. Roughly 350–450 characters with a handful of notes.
Encoding a link works with no signal.

`done` and `note` carry separate timestamps, so a merge takes the newer of the two per field
instead of one phone clobbering the other. Notes written before this existed have no
timestamp and count as oldest, so an incoming value wins. Unchecking travels too — an item
that was checked and then cleared still ships its timestamp. The link is dropped from the URL
as soon as it's read, so refreshing doesn't ask twice.

It is not automatic. Someone has to press the button, and nothing syncs in the background.

**On iPhone this is worth knowing:** a home-screen web app has its own storage, separate from
Safari. Open a share link in Safari and the merge lands in Safari, not in the installed app.
Pick one and stick with it. Android doesn't split them.

## Offline

The service worker precaches the page, the icons, and the fonts on install, so one load
on wifi is all it takes — after that it opens in a parking deck or a packed rooftop where
service drops. There are no third-party requests at all; the fonts are served from `fonts/`.

## Calendar

The **Add to calendar** button builds a `.ics` for the nine fixed-time items (both flights,
the five signature events, hotel checkout, and the car return) with an alert on each, and hands it to your
calendar app. It's generated in the browser from the same `DAYS` data the cards render
from, so it works offline too.

Each of those nine items carries an `ics:{...}` field right next to its `time:` on the
card. **Change a time on a card and change it in `ics` too** — they sit on the same object
so it's hard to miss, but nothing enforces it.

Times use `TZID` with real `VTIMEZONE` blocks rather than UTC, which is what lets the two
flights carry a Central departure and an Eastern arrival (and the reverse coming home).
The `UID`s are stable, so re-importing updates the events instead of duplicating them.

Only She Smokes Too publishes an end time (2:00 AM). The other four events' end times are
estimates and say so in their descriptions. The Block Party ends at 5:15 PM on purpose —
that's when you have to leave for the airport, not when it wraps.

Two of the bookings disagree with the flights, and the itinerary works around both rather
than pretending otherwise: the car is booked from 12:30 PM Thursday but you land at 12:41,
and it's booked until 9:00 PM Sunday but the flight leaves at 8:15. The Sunday card treats
6:00 PM as the real return deadline.

## After you edit index.html

Bump the cache name in `sw.js` (`cigarweek-v1` → `cigarweek-v2`) and push. Without that, phones
that already installed it may keep serving the old copy.

## Notes

- Checklist and planning notes save to `localStorage` on the device, and move
  between phones only when you press **Share my notes**.
- Dates are hardcoded to October 1–4, 2026 — the countdown and the auto-selected day key off it.
- Checklist state is keyed `atl2026:plan`; the New York trip's `nyc2026:plan` is left alone and ignored.
- No analytics and no network calls at all — nothing leaves the device.
