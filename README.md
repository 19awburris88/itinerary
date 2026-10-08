# The Guide — a personal calendar and travel itinerary

Static PWA. No build step, no dependencies, one hand-written file. Push it as-is.

```
index.html      the whole app (~2,700 lines: CSS, data, logic)
manifest.json   name, colors, icons
sw.js           offline cache
icons/          192 / 512 / maskable / apple-touch
fonts/          6 self-hosted woff2, latin subset
.nojekyll       stops GitHub Pages running Jekyll over it
```

## Three views

- **Today** — a concierge page. Greeting, On Now or Up Next with a progress bar and a
  leave-by, the rest of the day, what's protected, the next trip, and one concierge note.
- **The Guide** — the week as television programming. Work, Life, Wellness and Travel are
  channels along a shared timeline; events are program blocks, so overlaps and open time
  are visible at a glance. A red line marks now.
- **Trips** — a cover per trip, and inside it the day-by-day rail with flights, hotels,
  reservations, the calendar export and the checklist.

A sidebar appears at 900px and up; below that the brand and tabs sit on top.

## Where the data lives

Two sources, merged at read time by `eventsOn(date)`:

- **Trips** are hand-authored in the `TRIPS` array in `index.html`. They carry confirmation
  numbers, map links and `ics` blocks, and a deploy can never touch them.
- **Your own events** live in `localStorage` under `guide:events`, created and edited in the
  app with the **+** button. A deploy can never touch those either.

Repeats (daily, weekdays, weekly on chosen days, monthly) are expanded when a view asks for
a date range — never stored — so there are no duplicate rows to clean up when a series
changes. `offair` marks protected time: it shows as a dashed block and is skipped by Up Next.

Trip items join the timeline when they have an `ics` block or a parseable clock. "Morning",
"Day" and "TBA" stay untimed rather than being invented into a slot.

## Routing

`#today` · `#guide` · `#trips` · `#trip/<id>`. Older `#<tripid>` links, including shared
checklist links, still resolve.

## Themes

`data-theme` on the root plus one block of tokens per theme — **ivory** (warm paper,
charcoal, deep burgundy, brass) and **lounge** (the same room after dark). Every colour in
the app comes from those tokens, so adding a third theme means adding one block. Each trip
tints the app with its own accent on top: the darker value on ivory, the brighter one in
the lounge. The moon button switches, and the choice is remembered.

## Install

**Android** — open the URL in Chrome, menu (⋮) → **Install app**, launch from the home screen.

**iPhone** — open in Safari, Share → **Add to Home Screen**. Note that a home-screen web app
gets its own storage, separate from Safari, so events and checklists do not cross between
the two. Pick one and stay in it.

## Offline

The service worker precaches the page, icons, and fonts on install, so one load on wifi is
all it takes. No third-party requests at all.

## Calendar

**Add to calendar** on any trip builds a `.ics` for that trip's fixed-time items with an
alert on each, and hands it to your calendar app. Times use `TZID` with real `VTIMEZONE`
blocks so the flights keep two timezones each — Central out of DFW, Eastern in Atlanta and
Indianapolis, and Mexico City's fixed offset (no DST there since 2022). Generated in the
browser, works offline.

Event ids are `<trip uid>-<item uid>@19awburris88.github.io`, stable across regenerations,
so re-importing updates the events instead of duplicating them.

## Sharing a checklist

**Share my notes** packs the current trip's checklist into a link and hands it to the share
sheet. The other phone opens it, lands on the same trip, and gets a **Shared plan** panel
listing only what it doesn't already have. `done` and `note` carry separate timestamps and
merge newest-wins per field. A link for a trip you're not looking at switches you to it first.

On iPhone a home-screen web app has its own storage, separate from Safari — open share links
in whichever one you actually use.

## Adding a trip

Add an object to `TRIPS`, bump `CACHE` in `sw.js`, push. The Trips view picks it up, sorted
by date, with wrapped trips sinking to the bottom, and its days join The Guide automatically.

## After you edit index.html

Bump the cache name in `sw.js` (`guide-v2` → `guide-v3`) and push. Without that, phones that
already installed it keep serving the old copy. This is the main way this project breaks.

## Notes

- Checklists save per trip under `localStorage` keys `trips:<id>:plan`. The Atlanta list from
  before there were multiple trips is carried across automatically.
- Dates drive everything — nothing is hardcoded to a month any more.
- No analytics and no network calls at all.
