---
title: "README rewrite and documentation vault"
date: 2026-09-03
branch: docs/readmeUpdate
head: 21fc4e8
tickets: [Readme_And_Documentation_Vault]
tags: [session]
---

# README rewrite and documentation vault

> Written before the 2026-09-29 vault restructure. Folder and note names below are the originals
> (`10-Architecture/`, `Known-Issues`, …); links have been updated to the new names. Several findings
> here were later fixed by the refactor; see [[Known_Issues]] → "Resolved by the Refactor".

The repo's `README.md` was two lines — a title and a one-sentence description — so nobody arriving at
the project could learn what it was or how to run it. This session replaced it with a full README
including C4 diagrams, and created this Obsidian vault as the place the project's long-form
documentation and session history will live from now on.

No application code was touched. `src/`, `server/` and `email-service/` are unchanged.

---

## What Changed

| File | Status | What changed |
| --- | --- | --- |
| `README.md` | modified | Rewritten in full: overview, four Mermaid C4 diagrams (context, container, and component views of both the SPA and the API), key-component tour, three-terminal setup, known rough edges, scripts, annotated layout, deployment |
| `coaching-dashboard-documentation/` | created | Obsidian vault — 20 notes plus `.obsidian/` config |
| `coaching-dashboard-documentation/00-Home.md` | created | Map of content; entry point |
| `coaching-dashboard-documentation/10-Architecture/*.md` | created | `C4-Context`, `C4-Container`, `C4-Component`, `Data-Model`, `Auth-and-Roles`, `Frontend-Data-Flow` |
| `coaching-dashboard-documentation/20-Components/*.md` | created | `Coach-Dashboard`, `Team-Insights`, `API-Endpoints`, `Charts-and-Styling`, `Email-Service` |
| `coaching-dashboard-documentation/30-Operations/*.md` | created | `Local-Setup`, `Environment-Variables`, `Deployment`, `Testing`, `Known-Issues` |
| `coaching-dashboard-documentation/90-Meta/*.md` | created | `Vault-Conventions`, `How-To-Update` |
| `coaching-dashboard-documentation/40-Sessions/2026-09-03-...md` | created | This note |
| `.claude/commands/session-log.md` | created | The `/session-log` slash command |
| `.gitignore` | modified | Added `coaching-dashboard-documentation/.obsidian/workspace*.json` |
| `CLAUDE.md` | modified | New "Documentation vault" section pointing at the vault and `/session-log` |
| `AGENTS.md` | modified | New "Further documentation" section; kept as the source of truth for stack and conventions |

## Decisions and Why

- **Mermaid `flowchart` styled as C4, not Mermaid's native `C4Context` blocks.** Native C4 syntax
  frequently fails to render on GitHub, which would have left the README's main asset showing a parse
  error. Styled flowcharts render on GitHub, in Obsidian, and in VS Code preview. All 16 diagrams
  (4 in the README, 12 in the vault) were verified by rendering them with `@mermaid-js/mermaid-cli`.
- **Vault at the repo root, committed.** Session logs then land in git history alongside the code they
  describe, and teammates and other agents can read the architecture notes.
  `.obsidian/workspace*.json` is ignored so per-user pane layout does not churn the diff.
- **`/session-log` as an explicit command rather than a `Stop` hook.** A hook would fire on every
  turn, including trivial ones, and produce thin notes. An explicit command produces a real writeup
  when the work is actually finished.
- **Three documents, three lanes.** `README.md` = what and how to run; `AGENTS.md` = conventions, and
  still the source of truth; the vault = why, and what we did. They cross-link rather than duplicate.
  The one accepted duplication is the C4 diagrams, called out in [[README|Vault conventions]].
- **Documented the rough edges rather than hiding them.** A README that omits "`nodemailer` isn't in
  `package.json`" costs the next person an hour.

## Findings From Verifying Claims Against the Source

Writing the docs meant reading the code, which surfaced several things not previously written down:

- **`public/_redirects` never reaches the build.** Netlify publishes `src/dist`, but Vite's
  `publicDir` resolves to `src/public`, so the repo-root `public/` is not part of the build. Confirmed
  by a clean `npm run build`: no `_redirects` in the output. The live site's SPA fallback may be
  broken — worth a hard-refresh test on a deep link. Recorded in [[Deployment]] and [[Known_Issues]].
- **A cross-platform build discrepancy.** Windows is case-insensitive, so `src/public` resolves to the
  existing `src/Public/` image directory and Vite copies those images to the output root — something
  that will not happen on Netlify's Linux builder. The local build output is not identical to the
  deployed one.
- **`/api/team/heart-rate` does not share `buildTeamSeries`.** `AGENTS.md` states all four team
  endpoints share it; the heart-rate route in fact has its own `fetchAthleteHeartRows`, parallel
  fetching, a different label fallback, a `bpm` key instead of `value`, and 60-second-bin averaging
  instead of exact-timestamp matching. Documented in [[API_Endpoints]]. `AGENTS.md` was left alone —
  the correction lives in the vault rather than being edited into a file that other agents treat as
  canonical without context.
- **A fourth subcollection, `RespiratoryRateDataHX`**, exists and is read by the team endpoint;
  `AGENTS.md` lists only three. Added to [[Data_Model]].
- **The API verifies no auth token at all.** Any caller can read any athlete's data, enumerate all
  users, or grant themselves admin. This is now the top entry in [[Known_Issues]].
- **`HX` is Hexoskin** (from a comment in `server/server.js` and a tooltip in `V02MaxChart.jsx`);
  **`HC` is unexplained** anywhere in the repo. Recorded as unverified rather than guessed at.
- **The build is one 2.26 MB JS chunk** (662 KiB gzipped) with no code splitting.
- **`@tanstack/react-query`, `apexcharts` and `react-apexcharts` have no imports** in `src/`.

## Open Threads

- The `_redirects` question needs a check against the live Netlify URL — this repo cannot tell you
  whether the dashboard settings compensate.
- `node_modules` was absent from this working copy; `npm install` was run to verify the build. It is
  gitignored, so it does not appear in `git status`.
- The vault's `/session-log` command has not been exercised yet — this note was written by hand to
  the shape the command specifies. First real run will be the test.
- Nothing was committed. The git-guard hook forbids it and that is correct: review and commit are yours.

## Notes Updated

All vault notes are new in this session, so nothing was corrected. [[Deployment]] and
[[Known_Issues]] were amended after the build finished, to replace a predicted `_redirects` result
with the verified one.

---

## Related Notes

- [[README|Vault home]]
- [[03-Container]]
- [[API_Endpoints]]
- [[Known_Issues]]
- [[Readme_And_Documentation_Vault]]
