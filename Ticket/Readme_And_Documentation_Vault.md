---
title: "README Rewrite and Documentation Vault"
date_started: 2026-09-03
date_finished: 2026-09-03
repo: coaching-dashboard
area: docs
branch: docs/readmeUpdate
pr: "#32"
commit: 21fc4e8
status: merged
standups: []
skills: [technical-writing, c4-model, mermaid, code-auditing]
tags: [ticket]
---

# README Rewrite and Documentation Vault

Replaced a two-line README with a full one (overview, four C4 diagrams, setup, known rough edges), and created this Obsidian vault as the project's long-form documentation.

---

## Ticket

Pre-handoff documentation for the incoming capstone team. The README merged to `main` as PR #32; the vault is local (gitignored). Session write-up: [[Session_2026-09-03_Readme_And_Vault]].

## What I Did

- Rewrote `README.md`: overview, Mermaid C4 context/container/component diagrams, key-component tour, three-terminal setup, known rough edges, scripts, layout, deployment.
- Created the vault (20 notes), the `/session-log` command, and pointers from `CLAUDE.md` and `AGENTS.md`.
- Verified claims against the source rather than writing from memory, which surfaced findings nobody had written down.

## Technical Details

Findings from reading the code while documenting it, several later fixed by the refactor:

- `public/_redirects` never reached the Netlify build (confirmed by a clean build). Fixed in [[Refactor_Phase_2_Vite_Layout]].
- `/api/team/heart-rate` does not share `buildTeamSeries`, contrary to `AGENTS.md`. See [[API_Endpoints]].
- A fourth subcollection, `RespiratoryRateDataHX`, not listed in `AGENTS.md`.
- **The API verifies no auth token at all.** Became [[SEC_1_API_Auth_Tokens]].
- A 2.26 MB single JS chunk; three installed-but-unused packages (removed in Phase 1).

Chose Mermaid flowcharts styled as C4 over native `C4Context` blocks, because the native syntax often fails to render on GitHub.

## Testing

- All 16 diagrams rendered with `@mermaid-js/mermaid-cli`.
- `npm run build` on a clean tree to verify the `_redirects` claim.

## Impact

The first written account of how the system works, and the audit that seeded the refactor and the security tickets.

## Resume Bullets

- **Wrote architecture documentation with C4 diagrams** for a three-service React/Express/Firebase application ahead of a team handoff, verifying every claim against the source code.
- **Surfaced an unauthenticated API exposing athlete health data** during a documentation audit, and scoped the fix as a security ticket.

## Follow-ups / Open Questions

- The README is now pre-refactor; refreshing it is refactor Phase 7.

---

## Related Notes

- [[Session_2026-09-03_Readme_And_Vault]]
- [[Vault_Restructure]]
- [[Refactor]]
