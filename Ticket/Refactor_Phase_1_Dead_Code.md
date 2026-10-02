---
title: "Refactor Phase 1: Delete Dead Code and Dependencies"
date_started: 2026-09-05
date_finished: 2026-09-05
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: b539ba3
status: committed
standups: []
skills: [dependency-management, dead-code-removal, react, express]
tags: [ticket]
---

# Refactor Phase 1: Delete Dead Code and Dependencies

Deleted orphan files, commented-out prior versions and unused packages so that everything left in the tree is live code, and declared the one dependency that was missing.

---

## Ticket

First code phase of the pre-handoff refactor (`docs/refactor-phases.md`). Goal: make dead code distinguishable from live code before restructuring anything. See [[Refactor]] problem 2.

## What I Did

- Deleted 10 orphan files: three `*New` chart rewrites, `CoachDashboard2`, `Header`, `SleepGraph`, `TeamRosterCard`, `useCoachName`, a 0-byte `DatePicker/index.js`, `DateWindowPicker`; plus a stray `src/package-lock.json` and 3.2 MB of unreferenced images.
- Stripped commented-out prior versions: `SleepTimeline.jsx` 1,170 → 231 lines, `vite.config.js` 93 → 32, `sendInvitation.jsx` 157 → 83, `server/server.js` 713 → 642 (including the dead `POST /updateRole` and `GET /getUsers`, duplicated verbatim by their `/api/` twins).
- Removed 20 unused packages and added the undeclared `nodemailer`. `node_modules`: 989 → 547 packages. Declared deps 38 → 29, devDeps 25 → 15.
- Moved daisyUI removal here from Phase 5 (removing the package without its plugin line breaks the build).

## Technical Details

The original audit was wrong on four counts, caught during verification: `moment` is used in `SleepTimeline.jsx`'s live block; `date-fns` is a required peer of `chartjs-adapter-date-fns`; and two line-range estimates were off. Every deletion was re-checked by grep rather than trusted from the audit.

| File | Change |
|------|--------|
| 10 orphan components and pages | deleted |
| `SleepTimeline.jsx` | 1,170 → 231 lines |
| `server/server.js` | 713 → 642 lines, two legacy routes removed |
| `package.json` | −20 packages, +`nodemailer` |

## Testing

- `npm run build` green (2,538 modules); `node --check` on both edited server files; greps confirm no dangling references.
- Not verified: runtime click-through of API-backed pages.

## Impact

`node_modules` shrank from 989 to 547 packages. The email service starts after a clean `npm install` for the first time.

## Resume Bullets

- **Audited and removed dead code** from a React/Express codebase before a team handoff: 10 orphan files, 20 unused packages and ~1,100 lines of commented-out code, cutting installed packages from 989 to 547 with no behaviour change.
- **Caught four errors in the original audit** by re-verifying each deletion against the source instead of trusting the plan.

## Follow-ups / Open Questions

- The stripped code showed the invitation email POST was already commented out. See [[Email_Service]].

---

## Related Notes

- [[Refactor]]
- [[Refactor_Phase_2_Vite_Layout]]
- [[Tech_Stack]]
