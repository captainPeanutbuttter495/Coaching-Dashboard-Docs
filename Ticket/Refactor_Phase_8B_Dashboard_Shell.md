---
title: "Refactor Phase 8B: Coach Dashboard Shell and Header"
date_started: 2026-09-07
date_finished: 2026-09-07
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: 1374879
status: committed
standups: []
skills: [tailwind, css, chrome-devtools-protocol, visual-regression-testing]
tags: [ticket]
---

# Refactor Phase 8B: Coach Dashboard Shell and Header

Converted the coach dashboard's page shell, header and account menu to utilities, and proved the card grid had always been two columns at every width.

---

## Ticket

Refactor Phase 8, second step. See [[Refactor]] and [[Coach_Dashboard]].

## What I Did

- `style.css` 1,717 → 1,354 lines (363 removed). Only two files carried the classes: `pages/CoachDashboard.jsx` and `CoachDashboardHeader.jsx`.
- Dropdown items became a colourless base string plus mutually exclusive `Normal` / `Danger` colour strings, so no two utilities of one property compete.
- Kept `DashboardKpisRow` and `DashboardCard` out of scope: they share an inline-style object with the 12 cards, and converting half would split the seam.

## Technical Details

- **The grid is faithful, not fixed.** Its responsive rules were declared before an unconditional two-column rule, so they never applied. The dumps show the same `grid-template-areas` at all five widths. The dead rules were deleted and the rendered result kept; making it collapse is a design decision.
- Three traps caught before editing: `items-start` emits `flex-start`, not `start`; `calc()` in a class needs escaped spaces; blockification makes `inline-flex` compute as `flex`.
- Found that a Firestore quota outage, not the conversion, had emptied the Team Insights athlete list, and proved it with the stubbed capture states.

## Testing

- Computed-style dumps: **105/105 identical.** Noise floor 0.
- Region pixel diff of the converted band: **0 differing pixels at all five widths.**
- Hover states forced through CDP `CSS.forcePseudoState`: **7/7 match** the deleted declarations.
- Build 2,553 modules; CSS 41.37 → 40.68 KiB. `npx vitest run` 5/5.

## Impact

The dashboard shell is utilities, and the "why does it overflow below 900px" question has a measured answer.

## Resume Bullets

- **Verified hover and pseudo-state styling programmatically** by forcing states through the Chrome DevTools Protocol and comparing computed values to the deleted CSS (7/7 match).
- **Diagnosed a Firestore outage** (the daily read quota exhausted by unbounded queries) that initially presented as a CSS regression, and documented the root cause.

## Follow-ups / Open Questions

- Card grid overflow below ~900px; unbounded Firestore reads. See [[Known_Issues]] #4 and #17.

---

## Related Notes

- [[Refactor]]
- [[Coach_Dashboard]]
- [[API_Endpoints]]
