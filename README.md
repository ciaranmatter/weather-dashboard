# Live weather & tide dashboard

## What this is
A multi-location dashboard showing live weather, rainfall forecast, and modelled sea level, with front-page alert badges and a multi-site map view. Currently covers Melbourne, Sydney, Adelaide, Perth, and Brisbane, with a front page to pick a location and a detail view for live conditions. Built as a simple, zero-backend first step, part of a larger idea for Matter to eventually build flood-risk warnings. Company branding is intentionally left off the page itself for now.

Live at: **https://ciaranmatter.github.io/weather-dashboard/**

## How it works
- One static HTML file (`index.html`), no build step, no server.
- Front page shows a card per location (Melbourne, Sydney, Adelaide, Perth, Brisbane) with a live snapshot; clicking one opens the full detail view for that city.
- Fetches directly from the browser, per location:
  - Weather + rainfall forecast: Open-Meteo forecast API (BOM ACCESS-G model), free, no API key.
  - Sea level: Open-Meteo Marine API, free, no API key. Modelled, not an official BOM tide table — coastlines on an enclosed bay/gulf (Melbourne/Port Phillip Bay, Brisbane/Moreton Bay, Adelaide/Gulf St Vincent) are a rougher estimate; open coastlines (Sydney, Perth) tend to be more reliable.
- Adding another location = adding one entry to the `LOCATIONS` list in the script (name, weather coordinates, a coastal point for tide data).
- Charts drawn with Chart.js; the map view uses Leaflet with OpenStreetMap tiles — both loaded from a CDN, both free with no API key or account, both pinned with Subresource Integrity (SRI) hashes so the page only runs the exact script/stylesheet bytes it expects.
- Hosted on GitHub Pages, deployed automatically by pushing to `main`.
- Note: live data won't load in Claude's in-chat preview — that sandbox blocks calls to outside APIs for security. It works correctly once opened as a real file or hosted on GitHub Pages (confirmed working).

## Alerts (Melbourne only so far)
- Front-page badges and a detail-view "Alerts" card show when conditions cross a threshold:
  - **Heavy rain** — any forecast hour ≥ 4mm, or ≥ 25mm over 24h (from the existing Open-Meteo hourly forecast).
  - **Storm** — WMO weather codes 95/96/99 in the next 24h.
  - **High tide expected** — modelled sea level forecast to peak ≥ 0.5m in the next 48h. This threshold is a starting point — tune it once there's been time to see it run against real conditions.
- **High river**, **local flooding**, and a free-text **other** alert are manual toggles (no live source wired up yet) — set by a person, saved to `localStorage`, clearly labelled "(manually set)" wherever they show up so it's honest about what's live data vs. judgement call. Only persists on one browser/device until there's a real backend.
- Enabling alerts for another location is one config entry: `alerts: { enabled: true, tideThreshold: <meters> }` on that location's object.

## Map view (Melbourne, Sydney, Adelaide so far)
- Opening one of these cities defaults to a map (Leaflet + OpenStreetMap, no API key) with 4 pins — 2 rainfall sites and 2 tide sites per city — rather than one aggregated view for the whole city.
- Rainfall pins are droplet-shaped in the app's blue; tide pins are wave-shaped in the app's pink, so the two types are tellable apart by shape as well as color.
- Clicking a pin fetches and shows that specific site's current reading plus its own short-term chart, live for that site's exact coordinates (not just the city's primary reading).
- A "View full trends →" button switches to the original conditions-cards + full 48h charts view (still based on each city's primary CBD/tide-site reading) and back, so that view isn't lost, just secondary.
- No pin exists yet for river level — there's no live source for it, and an empty map is more honest than a pin showing fake data.
- Enabling the map for another location is one config entry: `map: { enabled: true, sites: [...] }` on that location's object, where each site is `{ type: 'rainfall' | 'tide', label, lat, lon }`.

## Design
- Custom design system (not a template): navy/royal-blue/pink brand palette on a charcoal-and-grey neutral base, Space Grotesk (headings) + Inter (body) + IBM Plex Mono (numeric readings), inspired by field survey / gauge-board aesthetics.
- Dark mode supported automatically via `prefers-color-scheme`.

## Where this came from
This started as a bigger brainstorm: Matter has IoT sensors on stormwater pits and bridges, and the original idea was a full flood-warning system combining live rainfall, river levels, tides, and Matter's own sensor data, with Claude interpreting fused data into plain-language warnings. That full version needs a real backend, multiple paid/free external data sources, and more infrastructure than made sense as a first build.

This dashboard is the deliberately small first step: prove out a live-data dashboard end to end, with only one data source and zero backend, before adding complexity back in. Alerts and the map view are the next layer on top of that same foundation, still with no backend.

## Scaling: what's safe to add now vs what needs a backend

**Stays entirely in the browser, no architecture changes needed:**
- More locations, and alerts/map for those locations — same one-entry-per-location config pattern already in place.
- More free, key-free live metrics from the same API family (UV index, wind gusts, air quality, etc.).
- Simple client-side math on the data already fetched (e.g. flagging today's forecast as high/low).

**Needs a small backend before attempting:**
- Any paid or key-locked API (e.g. WillyWeather for better tide accuracy, or a real river-level feed like Victoria's WMIS) — an API key embedded in a public page's source is visible to anyone, so keyed APIs must be called from a server that holds the key privately, not from the browser directly.
- Matter's own private sensor data (stormwater pits, bridges) — same reasoning; private data can't be fetched straight into a public page.
- Any history or trend analysis ("how has this changed over the past week", "flag when multiple signals agree") — the current dashboard has no memory between visits; this needs somewhere to store past readings and logic to run over them.
- Sharing manual alert toggles across visitors/devices — they currently live in one browser's `localStorage`; anything shared needs a real datastore, at which point the toggles should also gain some access control since anyone can currently flip them.
- Claude-powered analysis or warning text — same key-exposure issue as paid APIs; an Anthropic API key can't live in public page source either.

**When that threshold is hit**: the backend discussed earlier (Supabase — Postgres database + auto-generated API + mobile-ready client libraries, free to start) is the planned next layer. This dashboard becomes the frontend; nothing built so far needs to be thrown away.

## Immediate next steps
- Watch the heavy-rain/storm/tide alert thresholds against real conditions for a bit and tune them if they're too sensitive or not sensitive enough.
- Decide if/when manual alert toggles need real access control, before they're ever backed by something shared.
- Extend alerts to Sydney and Adelaide, and the map view to Perth and Brisbane, once the current versions have been used for a while.
- Revisit the Supabase-based architecture once ready to add private sensor data, history, river-level data, or Claude-generated analysis.

## Constraints to keep in mind
- No backend — keep data sources to ones with public, CORS-friendly, key-free APIs where possible, until the scaling threshold above is hit.
- Minimal cost — free tiers only until there's a clear reason to pay for something.
- Should stay easy to eventually wrap as a mobile app — keep logic/data-fetching cleanly separated from display where possible.
- Map tiles come directly from OpenStreetMap's public tile servers, which is fine at this traffic level — if this ever gets serious public usage, OSM's tile usage policy asks heavy-traffic sites to move to a hosted tile provider instead (e.g. MapTiler, Stadia).
