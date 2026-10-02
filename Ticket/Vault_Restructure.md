---
title: "Restructure the Vault to Match VIP-Docs and Boracle-Docs"
date_started: 2026-09-29
date_finished: 2026-09-29
repo: coaching-dashboard
area: docs
branch: refactor
pr:
commit:
status: in-review
standups: []
skills: [technical-writing, c4-model, mermaid, documentation-systems]
tags: [ticket]
---

# Restructure the Vault to Match VIP-Docs and Boracle-Docs

Reorganized this vault into the same shape as the owner's other two project vaults, refreshed every note against the post-refactor code, and added a `coaching-dashboard-docs-sync` skill to keep it current.

---

## Ticket

Owner request, 2026-09-29: match the structure and format of `VIP-Docs` and `Boracle-Docs`. Decisions made at planning: keep the vault in the repo folder (gitignored); keep `Sessions/` and add `Ticket/` + `Standup/`; create a sync skill.

## What I Did

- Replaced the numbered folders (`10-Architecture/` … `90-Meta/`) with `Project/`, `Frontend/`, `Backend/`, `Infrastructure/`, `SystemDesign/C4-Diagrams/`, `Sessions/`, `Standup/`, `Ticket/`, `_Templates/`, and `README.md` as the entry point.
- Renamed every note to `PascalCase_Underscores` and rewrote every wiki-link.
- Expanded C4 from three levels to the full `00`–`07` set (landscape, code level, dynamic diagrams are new) on the shared Mermaid palette.
- Added `Tech_Stack`, `Project_Structure`, `Roadmap`, one ticket note per refactor phase, `Resume_Highlights` and `Project_History`.
- **Re-verified every carried-over note against `refactor @ 99a9eb6`.** The originals predated most of the refactor.

## Technical Details

Corrections found while re-verifying (the old notes were wrong, not just renamed):

- The invitation flow does **not** call the email service; the POST was already commented out before Phase 1.
- `Admin.jsx` also fetches a bare `/api` path (the old note listed only `AthleteInfo` and `CoachDashboard2`).
- Paths throughout: `src/lib/`, `src/components/global/`, `src/features/*`, `.env.local` at the root, build to `dist/`.
- Known Issues: nine items fixed by the refactor moved to a "Resolved" table; items the refactor found (quota outage, silent failures, console logging, grid overflow) added.
- `Boracle-Docs` draws a dashboard → Expo push arrow that has no code in this repo; flagged in [[01-SystemLandscape]].

## Testing

- Every wiki-link resolves to exactly one file; every Mermaid block carries the init line and parses with `@mermaid-js/mermaid-cli`.
- Grep for the old folder and note names across the vault and repo docs.

## Impact

All three project vaults now read and are maintained the same way.

## Resume Bullets

- **Standardized documentation across three projects** into one vault structure with C4 diagrams, ticket histories and a sync workflow, re-verifying every note against the current code.

## Follow-ups / Open Questions

- Refactor Phase 7 (README, `AGENTS.md`, `PROJECT-READINESS.md`, `BACKLOG.md`) is still open. `AGENTS.md` still cites `src/Components/ProtectedRoute.jsx`.
- `tickets/BACKLOG.md` links point at the old vault paths.

---

## Related Notes

- [[Readme_And_Documentation_Vault]]
- [[README|Vault home]]
- [[Sessions/README|Session log]]
