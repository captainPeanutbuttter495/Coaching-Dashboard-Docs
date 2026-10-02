---
title: "Refactor Phase 3: Restructure Into Features"
date_started: 2026-09-06
date_finished: 2026-09-07
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: 4fa4ca3, d7443a9
status: committed
standups: []
skills: [react, architecture, code-organization]
tags: [ticket]
---

# Refactor Phase 3: Restructure Into Features

Reorganized `src/` from "grouped by kind" (`Components/`, `Pages/`, `Utils/`) into `pages/`, `features/<name>/`, `components/{ui,global}` and `lib/`, with every cross-directory import on the `@/` alias.

---

## Ticket

Refactor Phase 3. See [[Refactor]] problems 1 and 4, and [[Project_Structure]] for the result.

## What I Did

- Moved 47 files into the target tree and rewrote import lines in 32; no file body changed except import specifiers.
- Created `lib/`, `components/ui/`, `components/global/`, five features (`coach-dashboard`, `team-insights`, `athlete-health`, `invitations`, `settings`) and `pages/` with `pages/settings/`.
- Renamed `V02MaxChart.jsx` (leading **zero**) to `Vo2MaxChart.jsx`; `sendInvitation` and `fetchPendingInvitations` became `.js` (no JSX).
- Every cross-directory import is `@/…`; the only relative import left is `main.jsx → "./App"`.

## Technical Details

**The Windows case trap fired.** With `core.ignorecase=true`, git recorded the 14 case-only renames (`src/Pages/*` → `src/pages/*`) as in-place modifications and kept the capitalized paths in the index. That would have worked on Windows and broken the Linux Netlify build. The owner fixed the index in `d7443a9`, and Phase 6 later made wrong-case imports a lint error.

The build ran once at the end rather than after each group: mid-restructure the tree is never self-consistent, and one build resolves the whole module graph, which is the stronger check.

## Testing

- `npm run build` green at **2,539 modules**, identical to the Phase 2 baseline; CSS hash unchanged; JS identical in size.
- No `../` import left in `src/`; no reference to `Components/`, `Pages/`, `Utils/` or `V02MaxChart` survives.
- `npx vitest run`: 5 tests pass, proving the alias resolves under vitest.

## Impact

"Which files are the coach dashboard?" now has a one-folder answer.

## Resume Bullets

- **Restructured a React SPA into a feature-oriented architecture** (47 files moved, 32 import sites rewritten to a path alias) with a byte-identical build, as the foundation for a capstone team handoff.
- **Identified a git case-sensitivity hazard on Windows** that would have broken the Linux production build, and documented the fix.

## Follow-ups / Open Questions

- Pages with real logic (`Login`, `Register`, `Admin`, `AthleteAccess`) moved intact; splitting them is a follow-up.

---

## Related Notes

- [[Refactor]]
- [[Project_Structure]]
- [[04-Component-Frontend]]
