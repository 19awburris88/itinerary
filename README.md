# Austin's Trips — installable itinerary

Static PWA. No build step, no dependencies. One page holds every trip: a home screen lists
them, tapping one opens the day-by-day view with the flights, the stay, the events, a
calendar export, and the open questions for that trip.

```
index.html      the whole app — CSS in <style>, data + logic in <script>
manifest.json   name, colours, icons
sw.js           offline cache
icons/          192 / 512 / maskable / apple-touch
fonts/          5 self-hosted woff2, latin subset
```

Live at `https://19awburris88.github.io/itinerary/`. Push to `main` and Pages redeploys.

## How it's organised

Everything about a trip lives in one object in the `TRIPS` array:

| field | what it is |
|---|---|
| `id` | the URL fragment — `#atl` opens that trip |
| `uid` | prefix for calendar event ids; keep it stable so re-importing updates instead of duplicating |
| `name`, `h1`, `eyebrow`, `sub`, `meta` | the words on the home card and the trip hero |
| `start`, `end` | ISO dates; the countdown, the default day, and the home ordering all key off these |
| `theme` | `gold` (accent), `link` (the same accent dark enough for the light background), `ember` (event-card background) |
| `stay` | address, copy string, and the getting-there steps — or `null` to hide the section |
| `days` | one entry per day: `{d, title, tag, items[]}` |
| `todos` | the open items for the checklist |

`span(start, end, {date: day})` fills a date range with "wide open" days so a sparse trip
only spells out the days that matter. Each item has `{time, kind, title, where, cls, anchor,
facts[], note}`; `cls` is `session` (the highlighted event style) / `travel` / `meal` /
`open-slot`. Add `ics:{...}` to any item with a fixed time and it joins that trip's calendar
export — change a time on a card and change it in `ics` too.

## Routing

- no fragment → home, unless a trip is in progress today, in which case it opens straight to it
- `#trips` → home, always
- `#atl` → that trip
- `#atl&s=…` → that trip, with a shared checklist to merge

## Install on Android

1. Open the URL in Chrome.
2. Menu (⋮) → **Install app**.
3. Launch it from the home screen.

## Offline

The service worker precaches the page, icons, and fonts on install, so one load on wifi is
all it takes. No third-party requests at all.

## Calendar

**Add to calendar** on any trip builds a `.ics` for that trip's fixed-time items with an
alert on each, and hands it to your calendar app. Times use `TZID` with real `VTIMEZONE`
blocks so the flights keep two timezones each — Central out of DFW, Eastern in Atlanta and
Indianapolis, and Mexico City's fixed offset (no DST there since 2022). Generated in the
browser, works offline.

Event ids are `<trip uid>-<item uid>`, stable across regenerations.

## Sharing a checklist

**Share my notes** packs the current trip's checklist into a link and hands it to the share
sheet. The other phone opens it, lands on the same trip, and gets a **Shared plan** panel
listing only what it doesn't already have. `done` and `note` carry separate timestamps and
merge newest-wins per field. A link for a trip you're not looking at switches you to it first.

On iPhone a home-screen web app has its own storage, separate from Safari — open share links
in whichever one you actually use.

## Adding a trip

Add an object to `TRIPS`, bump `CACHE` in `sw.js`, push. The home page picks it up, sorted
by date, with wrapped trips sinking to the bottom.

## After you edit index.html

Bump the cache name in `sw.js` (`trips-v1` → `trips-v2`) and push. Without that, phones that
already installed it keep serving the old copy. This is the main way this project breaks.

## Notes

- Checklists save per trip under `localStorage` keys `trips:<id>:plan`. The Atlanta list from
  before there were multiple trips is carried across automatically.
- Dates drive everything — nothing is hardcoded to a month any more.
- No analytics and no network calls at all.
