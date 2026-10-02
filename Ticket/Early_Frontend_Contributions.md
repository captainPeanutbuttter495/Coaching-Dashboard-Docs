---
title: "Early Frontend Contributions (2024–2025)"
date_started: 2024-03-24
date_finished: 2025-03-10
repo: coaching-dashboard
area: frontend
branch: various (merged to main)
pr:
commit: 881d559 … 3a56d7a (10 commits)
status: merged
standups: []
skills: [react, firebase-auth, role-based-access, chart-js, mui]
tags: [ticket]
---

# Early Frontend Contributions (2024–2025)

My first ten commits on the dashboard, under the `captain.peanutbutter495` identity: the login UI, role-based navigation, the admin data grid, the first sleep and heart-rate charts, and the route tests.

---

## Ticket

Feature work on the original team, before the refactor. Reconstructed from `git log --author=captain` on 2026-09-29; there are no ticket write-ups from the time, so this note records only what the commits show.

## What I Did

| Date | Commit | What the commit shows |
|------|--------|-----------------------|
| 2024-03-24 | `881d559` | Gradient background on the home page |
| 2024-06-20 | `34a67ea` | Login UI refactor (11 files, +677/−564); added the ARCS mark |
| 2024-06-23 | `e3e87c8` | `Footer` component; login page update |
| 2024-06-26 | `a42a3ed` | Admin data grid (MUI) |
| 2024-06-29 | `2edb18e` | Role-based navigation: `Home` → `AthleteDashboard`, new `CoachDashboard`, role routing in `Login` |
| 2024-06-29 | `a9cd64d` | First sleep and heart-rate graphs and pages |
| 2024-06-30 | `9489ec7` | Fixed navigation access and redirection |
| 2024-07-19 | `f8e6e2e` | Activity ring; coach dashboard and invitation page work; `AthleteInfo` |
| 2025-03-10 | `ddb43da` | API route reading `HeartRateDataHC` from Firestore; sleep and heart pages on real data |
| 2025-03-10 | `3a56d7a` | `routes.test.jsx`: login → role → redirect test |

## Technical Details

Several of these survive in today's tree in refactored form: the role routing in `Login.jsx` (see [[Auth_And_Roles]]), `Footer`, `ActivityRing`, the Admin data grid, `AthleteInfo`, and `routes.test.jsx`. `SleepGraph` and the separate heart/sleep pages were later superseded and deleted as orphans in [[Refactor_Phase_1_Dead_Code]].

## Testing

- `routes.test.jsx` (2025-03-10). Its test credential has since been deleted from Firebase; see [[Testing]].

## Impact

The role-based navigation pattern, still in use, started here.

## Resume Bullets

- **Built role-based navigation and access** (admin / coach / athlete) for a React + Firebase coaching dashboard, including the login flow, redirects and an admin user grid.
- **Implemented the first athlete health charts** (sleep stages, heart rate, activity ring) and connected them to live Firestore data.

## Follow-ups / Open Questions

- Needs confirmation: which PRs these landed through. `git log --merges` does not attribute them clearly.

---

## Related Notes

- [[Project_History]]
- [[Auth_And_Roles]]
- [[Resume_Highlights]]
