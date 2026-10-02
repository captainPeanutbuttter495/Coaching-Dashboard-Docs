# Coaching Dashboard — Athlete Health Insights for Coaches

Documentation vault for the **Coaching Dashboard** (`coaching-dashboard`), a CSUN ARCS web app that aggregates athlete health data (heart rate, sleep, VO₂ max, respiratory rate) from Firestore and renders it for coaches. It is a React SPA on Netlify, an Express API on Render, and a small email service, all in one repo.

This vault lives **inside the repo folder** (`coaching-dashboard/coaching-dashboard-documentation/`) but is **gitignored**, so it exists only on the owner's machine. It follows the same structure as the `VIP-Docs` and `Boracle-Docs` vaults. The repo's `README.md` answers *what is this and how do I run it*; `AGENTS.md` holds the conventions; **this vault holds the why**, and what we did.

---

## Repositories

| Repo | Path | Stack | Role |
|------|------|-------|------|
| `coaching-dashboard` | `C:\Users\mbg91918\coaching-dashboard` | React 18 + Vite + Tailwind, Express + firebase-admin, nodemailer | **This vault** |
| `smart-clothing-app` | — | React Native, Expo, Firebase | Sibling: the athlete mobile app that writes the health data |
| `boracle-platform` | `C:\Users\mbg91918\boracle-platform` | Spring Boot 3, PostgreSQL, Firebase Auth | Sibling: ARCS data middleware. No integration yet |

All live under `github.com/CSU-Northridge-ARCS-Dev`. Work flows `feature-branch` → PR → `main`. The pre-handoff refactor runs on the long-lived **`refactor`** branch, one phase per session.

---

## Current State (as of 2026-09-29, `refactor @ 99a9eb6`, `origin/main @ 666ede1`)

- **Refactor Phases 0–6 and 8A–8D are committed** on `refactor` (9 commits ahead of `main`, not merged). Phase 7, the documentation refresh, is not started. See [[Refactor]].
- **Structure:** feature-oriented `src/` (`pages/` → `features/` → `components/` → `lib/`), enforced by ESLint. See [[Project_Structure]].
- **Styling:** Tailwind utilities only; `src/style.css` is three lines. See [[Styling_And_Charts]].
- **Lint:** 0 errors, 39 deliberate warnings. **Tests:** 5 live-Firebase smoke tests. See [[Testing]].
- **Live data:** only [[Team_Insights]] reads it through the deployed API. The [[Coach_Dashboard]] renders mock modules. All health data is stale (newest sample 2025-03-27).
- **Security:** the API verifies no token and Firestore rules allow any signed-in user everything. Planned: [[SEC_1_API_Auth_Tokens]], [[SEC_2_Firestore_Rules]].
- **Actively failing:** unbounded Firestore reads exhausted the daily quota on 2026-09-07. See [[Known_Issues]] #4.
- **The email service has no caller**; invitations write Firestore only. See [[Email_Service]].

---

## Documentation

### Quick Links

- [[Tech_Stack]]: technologies and versions
- [[Project_Structure]]: repo tree, layering rule, where this vault fits
- [[Roadmap]]: built vs. planned, refactor status
- [[Refactor]]: the pre-handoff refactor and the principle behind each phase
- [[Testing]]: what the tests do, and how the refactor was verified instead
- [[Known_Issues]]: verified defects and risks, plus what the refactor fixed

---

### Frontend

- [[Auth_And_Roles]]: the two independent role gates, and the refresh bug
- [[Frontend_Data_Flow]]: `apiUrl()` vs. bare paths, the dev proxy, UTC date windows
- [[Coach_Dashboard]]: the mock-data card grid and its synchronized 6.5s tick
- [[Team_Insights]]: the live-data page and the reference for new pages
- [[Styling_And_Charts]]: Tailwind conventions, measured traps, which chart library goes where

---

### Backend

- [[API_Endpoints]]: every route, the team-endpoint divergence, unbounded reads, no auth
- [[Data_Model]]: Firestore collections, DocumentReferences, inconsistent field names
- [[Email_Service]]: the invitation mailer nothing calls
- [[Configuration]]: every environment variable and what breaks without it

---

### Infrastructure

- [[Local_Dev_Setup]]: install, configure, run, get a role
- [[Deployment]]: Netlify + Render, the deploy checklist

---

### System Design

- [[00-C4-Overview]]: C4 index, palette, key decisions
- [[01-SystemLandscape]] · [[02-SystemContext]] · [[03-Container]]
- [[04-Component-Frontend]] · [[05-Component-API]] · [[06-Code]] · [[07-Dynamic-Diagrams]]

---

### Sessions

A dated log of work sessions, written by `/session-log`. See [[Sessions/README|Session index]].

---

### Standup

Meeting transcripts, prep notes and question lists, plus the open questions waiting for an answer. See [[Standup/README|Standup index]].

---

### Ticket

My work history on this project with resume-ready bullets. See [[Ticket/README|Ticket index]], [[Resume_Highlights]] and team [[Project_History]].

---

## Conventions

- File names use `PascalCase_Underscores` and **must be unique across the vault**, because wiki-links resolve by name. Folder indexes are `README.md` and are linked with their path (`[[Ticket/README|Ticket index]]`).
- C4 files are numbered `NN-Name.md`.
- Content notes have no frontmatter and follow one layout: `# Title`, a one-line summary, `---`-separated sections (Overview, domain sections, **Key Files**, **Tests**), then **Related Notes**. Ticket, Standup and Session notes carry frontmatter from their template in `_Templates/`.
- Diagrams are **Mermaid** (native in Obsidian). The palette and init line are in [[00-C4-Overview]]. Put a `### Legend` after every diagram.
- **Accuracy rule:** document what is on the `refactor` branch working tree. Anything planned but not built is labeled *(planned)* and drawn dashed. Record only real numbers; never invent metrics.
- **Domain notes say what is true now; session notes say what happened on a day.** If a change makes a domain note wrong, fix the domain note.
- The repo `README.md` carries copies of the context, container and component diagrams. Change one, change both.
- **What does not belong here:** secrets, service-account JSON, real athlete data, file listings git already records, and restatements of `AGENTS.md` (link to it instead).
- Code lives in the repo. Reference it by path; small illustrative snippets are fine.
- Claude keeps this vault in sync with the `coaching-dashboard-docs-sync` skill (`~/.claude/skills/coaching-dashboard-docs-sync/`) and the repo's `/session-log` command. Neither ever touches git state.
