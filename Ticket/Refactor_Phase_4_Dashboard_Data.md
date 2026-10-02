---
title: "Refactor Phase 4: Extract the Coach Dashboard's Static Data"
date_started: 2026-09-07
date_finished: 2026-09-07
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: 28a8a25
status: committed
standups: []
skills: [react, data-modeling, testing]
tags: [ticket]
---

# Refactor Phase 4: Extract the Coach Dashboard's Static Data

Lifted the mock athlete and team content out of the dashboard cards into 13 data modules, and gave each card a prop seam so a real API can replace the mock without touching the card.

---

## Ticket

Refactor Phase 4. See [[Refactor]] problem 5 and [[Coach_Dashboard]].

## What I Did

- Created 12 modules under `features/coach-dashboard/data/` plus `features/athlete-access/data/athletes.js`; rewired 14 files.
- Cards take the data as a **prop defaulting to the module**, so the page still passes nothing and the injection seam exists for the first time.
- Collapsed two hardcoded copies of the coach's name into `coachProfile`.
- Turned two cards' hand-written `<div>` lists into `.map()` over the new arrays.

## Technical Details

Only domain data moved; presentation config (formation slots, colour maps, axis domains, thresholds) stayed in the components by decision. `TeamLoadZonesCard` got **no** prop seam: its data is consumed in five nested helpers, and a partial seam would update the charts but not the summary beside them, which is worse than none.

## Testing

- All 12 cards rendered through `renderToStaticMarkup` before and after, asserted **byte-identical**.
- `npm run build` green at 2,552 modules (2,539 + 13 data modules); CSS hash unchanged. `npx vitest run` 5/5.
- Not verified: click-through of toggles, modals and drag-and-drop.

## Impact

The dashboard is one prop per card away from live data.

## Resume Bullets

- **Separated mock data from presentation** across 12 dashboard cards, adding prop-based injection seams for a future API, verified byte-identical with server-side rendering comparisons.

## Follow-ups / Open Questions

- Recorded eight real defects without fixing them (a fix would change what renders): the team builder's empty roster tabs, frozen "derived" values, 12-vs-18 athlete counts, inert Athlete Access buttons, and more. See [[Known_Issues]].

---

## Related Notes

- [[Refactor]]
- [[Coach_Dashboard]]
- [[Known_Issues]]
