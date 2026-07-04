[README.md](https://github.com/user-attachments/files/29656140/README.md)
# Matter — live weather & tide dashboard

## What this is
A single-page dashboard showing live weather, rainfall forecast, and modelled sea level for Melbourne / Port Phillip Bay. Built as a simple, zero-backend first step, part of a larger idea for Matter to eventually build flood-risk warnings.

## How it works
- One static HTML file (`index.html`), no build step, no server.
- Fetches directly from the browser:
  - Weather + rainfall forecast: Open-Meteo forecast API (BOM ACCESS-G model), free, no API key.
  - Sea level: Open-Meteo Marine API, free, no API key. Modelled, not an official BOM tide table — treat as approximate, especially near the bay entrance.
- Charts drawn with Chart.js (loaded from a CDN).
- Hosted on GitHub Pages, deployed by pushing to this repo.

## Design
- Custom design system (not a template): paper-blue palette, Space Grotesk (headings) + Inter (body) + IBM Plex Mono (numeric readings), inspired by field survey / gauge-board aesthetics.
- Dark mode supported automatically via `prefers-color-scheme`.

## Where this came from
This started as a bigger brainstorm: Matter has IoT sensors on stormwater pits and bridges, and the original idea was a full flood-warning system combining live rainfall, river levels, tides, and Matter's own sensor data, with Claude interpreting fused data into plain-language warnings. That full version needs a real backend, multiple paid/free external data sources, and more infrastructure than made sense as a first build.

This dashboard is the deliberately small first step: prove out a live-data dashboard end to end, with only one data source and zero backend, before adding complexity back in.

## Scaling: what's safe to add now vs what needs a backend

**Stays entirely in the browser, no architecture changes needed:**
- More locations — Open-Meteo supports multiple coordinates in a single request, so this is mostly just more UI, not more infrastructure.
- More free, key-free live metrics from the same API family (UV index, wind gusts, air quality, etc.).
- Simple client-side math on the data already fetched (e.g. flagging today's forecast as high/low).

**Needs a small backend before attempting:**
- Any paid or key-locked API (e.g. WillyWeather for better tide accuracy) — an API key embedded in a public page's source is visible to anyone, so keyed APIs must be called from a server that holds the key privately, not from the browser directly.
- Matter's own private sensor data (stormwater pits, bridges) — same reasoning; private data can't be fetched straight into a public page.
- Any history or trend analysis ("how has this changed over the past week", "flag when multiple signals agree") — the current dashboard has no memory between visits; this needs somewhere to store past readings and logic to run over them.
- Claude-powered analysis or warning text — same key-exposure issue as paid APIs; an Anthropic API key can't live in public page source either.

**When that threshold is hit**: the backend discussed earlier (Supabase — Postgres database + auto-generated API + mobile-ready client libraries, free to start) is the planned next layer. This dashboard becomes the frontend; nothing built so far needs to be thrown away.

## Immediate next steps
- Finish GitHub Pages deployment (repo created; clone locally, add `index.html` + this README, push, enable Pages in repo settings).
- Optionally add more locations or free metrics (see above) while still backend-free.
- Revisit the Supabase-based architecture once ready to add private sensor data, history, or Claude-generated analysis.

## Constraints to keep in mind
- No backend — keep data sources to ones with public, CORS-friendly, key-free APIs where possible, until the scaling threshold above is hit.
- Minimal cost — free tiers only until there's a clear reason to pay for something.
- Should stay easy to eventually wrap as a mobile app — keep logic/data-fetching cleanly separated from display where possible.
