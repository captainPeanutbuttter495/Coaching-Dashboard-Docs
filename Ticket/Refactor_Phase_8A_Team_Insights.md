---
title: "Refactor Phase 8A: Team Insights Converted to Tailwind"
date_started: 2026-09-07
date_finished: 2026-09-07
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: d9c5d92
status: committed
standups: []
skills: [tailwind, css, visual-regression-testing]
tags: [ticket]
---

# Refactor Phase 8A: Team Insights Converted to Tailwind

Converted the Team Insights page and its two graphs to utilities, and found that the breakpoint config added in Phase 5 was silently disabling Tailwind's `max-*` variants.

---

## Ticket

Refactor Phase 8, pulled forward ahead of Phases 6 and 7 by owner decision so the conversion lands before the handoff. See [[Refactor]].

## What I Did

- `style.css` 2,132 → 1,718 lines (63 rules, 4 `@media` blocks removed).
- Converted `pages/TeamInsights.jsx`, `MultiHeartGraph.jsx`, `SleepStageGraph.jsx`; new `graphClasses.js` for shared class strings.
- **Removed the `screens` config** and rewrote 40 `to*` tokens in 10 files as inline `max-[900px]:` variants.

## Technical Details

- Declaring `screens` in object form makes Tailwind disable the whole `min-*`/`max-*` family, warning once and then emitting nothing. Reordering named screens does nothing either: Tailwind sorts them by value, proven by byte-identical CSS (64,942 bytes) under both orders.
- Two conversion bugs caught by the dumps: `border-0` is not `border: 0`; and a disabled `<select>` makes *every* option `:disabled`.
- Legacy `rgba()` vs Tailwind's `rgb(r g b / a)`: identical computed strings, but Chrome rasterizes them one 8-bit step apart. Kept the original notation as arbitrary properties.

## Testing

- Computed-style dumps: **105/105 identical** (21 states × 5 widths, ~95 properties per element). Noise floor 0 across 105 files, measured twice.
- All 25 team-insights screenshots byte-identical. Build 2,553 modules; CSS 44.49 → 41.37 KiB.
- Measured that whole-page `/coach` screenshots are unusable evidence: 30 of 105 differ between two runs of identical code.

## Impact

Every breakpoint in the app now resolves in the right order, and the method's limits are measured rather than assumed.

## Resume Bullets

- **Found and fixed a silent Tailwind configuration failure** (object-form `screens` disabling all `max-*` variants) by building the CSS under controlled variations and diffing the output.
- **Quantified the reliability of screenshot testing** (30 of 105 captures differ on identical code) and switched the project's evidence to computed-style dumps with a zero noise floor.

## Follow-ups / Open Questions

- `PopupCalendar` injects its own global `<style>` block and shows no month label. See [[Known_Issues]] #24.

---

## Related Notes

- [[Refactor]]
- [[Team_Insights]]
- [[Styling_And_Charts]]
