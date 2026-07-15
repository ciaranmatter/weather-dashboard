# Live weather & tide dashboard

## What this is
A multi-location dashboard showing live weather, rainfall forecast, and modelled sea level, with front-page alert badges and a multi-site map view. Currently covers Melbourne, Sydney, Adelaide, Perth, Brisbane, and Ho Chi Minh City, with a front page to pick a location and a detail view for live conditions. Built as a simple, zero-backend first step, part of a larger idea for Matter to eventually build flood-risk warnings. Company branding is intentionally left off the page itself for now.

Live at: **https://ciaranmatter.github.io/weather-dashboard/**

## How it works
- One static HTML file (`index.html`), no build step, no server.
- Front page shows a card per location with a live snapshot; clicking one opens the full detail view for that city.
- Fetches directly from the browser, per location:
  - Weather + rainfall forecast: Open-Meteo forecast API (BOM ACCESS-G model for Australian cities), free, no API key.
  - Sea level: Open-Meteo Marine API, free, no API key. Modelled, not an official tide table — coastlines on an enclosed bay/gulf/river (Melbourne/Port Phillip Bay, Brisbane/Moreton Bay, Adelaide/Gulf St Vincent, Ho Chi Minh City/Saigon River estuary) are a rougher estimate; open coastlines (Sydney, Perth) tend to be more reliable. The Marine API can also return `null` for specific hours/sites it has no coverage for (seen at Ho Chi Minh City's Nhà Bè site, likely because a river point isn't well covered by the ocean model grid) — the app shows "—" rather than crashing when that happens.
- Adding another location = adding one entry to the `LOCATIONS` list in the script (name, weather coordinates, a coastal point for tide data).
- Charts drawn with Chart.js; the map view uses Leaflet with CARTO Voyager tiles (built on OpenStreetMap data) — both loaded from a CDN, both free with no API key or account, both pinned with Subresource Integrity (SRI) hashes so the page only runs the exact script/stylesheet bytes it expects.
- Hosted on GitHub Pages, deployed automatically by pushing to `main`.
- Note: live data won't load in Claude's in-chat preview — that sandbox blocks calls to outside APIs for security. It works correctly once opened as a real file or hosted on GitHub Pages (confirmed working).

## Alerts (all cities)
- Front-page badges and a detail-view "Alerts" card show when conditions cross a threshold:
  - **Heavy rain** — any forecast hour ≥ the rain threshold (default 4mm), or ≥ 25mm over 24h (from the existing Open-Meteo hourly forecast).
  - **Storm** — WMO weather codes 95/96/99 in the next 24h.
  - **High tide expected** — modelled sea level forecast to peak ≥ the tide threshold in the next 48h.
- Both thresholds are per-city and user-adjustable: click the bell icon at the top right of the front page to open **Alert thresholds**, which lists every alert-enabled city with a rain (mm/hr) and tide (m) input. Changes save to `localStorage` and apply immediately — badges, sorting, and the summary strip all recompute from the last-fetched data with no refetch needed.
- Defaults are 0.5m tide / 4mm rain for Melbourne, Sydney, Adelaide, and Perth; Brisbane defaults to 1.5m tide since its Moreton Bay readings run structurally higher (~1.2m vs ~0.1–0.6m elsewhere) — using the same 0.5m there would fire constantly. All defaults are just starting points; tune them via the bell icon once you've watched them run against real conditions for a bit.
- Enabling alerts for another location is one config entry: `alerts: { enabled: true, tideThreshold: <meters>, rainHourlyThreshold: <mm> }` on that location's object.

## Map view (all cities)
- Opening a city shows a two-column detail view: the interactive map (Leaflet + OpenStreetMap, no API key) on the left, fixed in place, and a scrollable data panel on the right — current conditions, a "Sites" list, and the full 48h charts. The map never moves as you scroll the right-hand panel. On narrow/mobile screens this stacks vertically instead (map on top, unfixed, details below in normal page flow).
- Each city has 4 pins — 2 rainfall sites and 2 tide sites — rather than one aggregated view for the whole city. Rainfall pins are droplet-shaped in the app's blue; tide pins are wave-shaped in the app's pink, so the two types are tellable apart by shape as well as color.
- The "Sites" list in the data panel mirrors the pins, showing each site's current key reading (rain amount or sea level) fetched eagerly on load. Clicking a site in the list pans the map to and opens that pin's popup (with its full reading plus a short-term chart); clicking a pin on the map highlights the matching list entry — either direction works.
- No pin exists yet for river level — there's no live source for it, and an empty map is more honest than a pin showing fake data.
- Enabling the map for another location is one config entry: `map: { enabled: true, sites: [...] }` on that location's object, where each site is `{ type: 'rainfall' | 'tide', label, lat, lon }`.
- Front-page cards also show a small static preview of that same map (same pins, smaller icons) — all interaction disabled (no drag/zoom/scroll), so it reads as an image; clicking anywhere on the card, including the preview map, opens the full interactive detail view. Fine for a handful of cities loading tiles up front; if the location list grows to a dozen+, worth revisiting lazy-loading these as they scroll into view.

## Front page: local time, sorting, and alert summary
- Each city panel shows its current local time (e.g. "2:34 PM local"), computed from its IANA timezone (e.g. `Australia/Adelaide`) so daylight saving is handled automatically without any manual offset math. Updates every 30s.
- Panels for cities with active alerts float to the front of the grid (most active alerts first), everything else keeps its normal order behind them.
- A one-line strip above the grid ("N cities have active alerts right now.") appears only when at least one alert is active anywhere, and is hidden entirely otherwise. Clicking it scrolls to and briefly highlights the affected panel(s).

## Design
- Custom design system (not a template): a blue-forward palette (light-blue page background, navy/royal-blue accent, pink for tide) on a charcoal-and-grey neutral base, Sora (headings) + Public Sans (body) + IBM Plex Mono (numeric readings), inspired by field survey / gauge-board aesthetics.
- Map tiles use CARTO Voyager (built on OpenStreetMap data, free, no API key) rather than default OSM tiles, for a cleaner, more legible basemap that leans into the blue-water theme. Pin icons (rainfall droplet, tide wave) are custom SVGs in the app's own colors.
- Page content runs wider than a typical article layout (1440px) so the location grid and detail view have more room to breathe on wide screens.
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
- Sharing alert threshold settings across visitors/devices — they currently live in one browser's `localStorage`; anything shared needs a real datastore.
- Claude-powered analysis or warning text — same key-exposure issue as paid APIs; an Anthropic API key can't live in public page source either.

**When that threshold is hit**: the backend discussed earlier (Supabase — Postgres database + auto-generated API + mobile-ready client libraries, free to start) is the planned next layer. This dashboard becomes the frontend; nothing built so far needs to be thrown away.

## Immediate next steps
- Watch the alert thresholds against real conditions for a bit and tune them (via the bell icon) if they're too sensitive or not sensitive enough — several cities are showing "High tide expected" at the current defaults, which is expected given how tide peaks fluctuate over a 48h window, but worth adjusting per city as real behavior becomes clear.
- Revisit the Supabase-based architecture once ready to add private sensor data, history, river-level data, or Claude-generated analysis.

## Constraints to keep in mind
- No backend — keep data sources to ones with public, CORS-friendly, key-free APIs where possible, until the scaling threshold above is hit.
- Minimal cost — free tiers only until there's a clear reason to pay for something.
- Should stay easy to eventually wrap as a mobile app — keep logic/data-fetching cleanly separated from display where possible.
- Map tiles come directly from OpenStreetMap's public tile servers, which is fine at this traffic level — if this ever gets serious public usage, OSM's tile usage policy asks heavy-traffic sites to move to a hosted tile provider instead (e.g. MapTiler, Stadia).
