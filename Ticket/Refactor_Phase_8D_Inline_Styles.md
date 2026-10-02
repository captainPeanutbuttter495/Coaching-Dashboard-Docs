---
title: "Refactor Phase 8D: Inline Style Objects Converted"
date_started: 2026-09-14
date_finished: 2026-09-14
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: 5ae5ae6
status: committed
standups: []
skills: [tailwind, codemods, visual-regression-testing]
tags: [ticket]
---

# Refactor Phase 8D: Inline Style Objects Converted

Converted the coach dashboard's inline `style={{}}` objects to Tailwind classes: 185 in scope → 23, where every one left is data-driven and cannot be a class.

---

## Ticket

Split from Phase 8C at planning. Scope: the coach dashboard only (`DashboardCard`, `DashboardKpisRow`, the 12 cards, `CoachDashboard.jsx`, `SyncedSegmentedToggle`). About 35 stragglers on other routes are a mop-up ticket. See [[Refactor]].

## What I Did

- Wrote a **converter** that encoded the earlier phases' rules (arbitrary px values, `rgba()` preserved, `[border:0]`, `rounded-[999px]`, never shorthand plus longhand) and returned anything it could not convert faithfully as residue rather than guessing.
- Ran it in four gated steps: 150 static objects; the `dashboardCardStyle` seam; 18 partially static objects; 13 boolean variants → mutually exclusive class strings.
- Moved every dynamic expression as source text, never retyped, so no behaviour could change silently.
- Deleted the dead `dashboardInnerPanelStyle` export.

## Technical Details

- The tracker's "~280" objects was a line count; the real figure was 220 repo-wide and 185 in scope.
- The 23 left are irreducible: data-driven colours and widths, a module constant, player coordinates, and CSS-variable-driven animation. "Everything a class can express is a class" is the accurate summary.
- `DASHBOARD_CARD_BASE` deliberately carries no padding, because two consumers override it and `p-[18px]` beside `p-[16px]` resolves by Tailwind's sort order.

## Testing

- **Step 0: all 195 generated classes compiled through the Tailwind CLI** and checked to emit CSS before any edit.
- Computed-style dumps: **115/115 identical at each of four steps.**
- Region pixel diffs with a per-card noise floor: 27 of 29 stable regions identical; the two outliers attributed to Chrome edge rasterization.
- Build 2,554 modules; JS slightly smaller (object literals gone). `npx vitest run` 5/5.

## Impact

The coach dashboard is styled one way. The pre-flight emission check is now part of the method.

## Resume Bullets

- **Built a codemod that converted 162 inline React style objects to Tailwind classes**, returning anything it could not convert faithfully as residue; verified with 115/115 identical computed-style captures at every step.
- **Added a pre-flight check** compiling all 195 generated classes to confirm each emits CSS, closing the failure mode that had silently deleted styling in the previous phase.

## Follow-ups / Open Questions

- `DashboardKpisRow` and `TeamInjuryRisk` still duplicate card chrome rather than rendering `<DashboardCard>`.
- The card grid overflow now blocks evidence: four cards fall outside the viewport at 800–900px.

---

## Related Notes

- [[Refactor]]
- [[Coach_Dashboard]]
- [[Styling_And_Charts]]
