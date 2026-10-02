---
title: "SEC-1: Require a Verified Firebase ID Token on Every API Route"
date_started: 2026-09-03
date_finished:
repo: coaching-dashboard
area: api
branch:
pr:
commit:
status: planned
standups: []
skills: [security, firebase-auth, express]
tags: [ticket]
---

# SEC-1: Require a Verified Firebase ID Token on Every API Route

*(planned)* Close the unauthenticated API: every route verifies a Firebase ID token and checks the caller's role and roster before serving.

---

## Ticket

Planned 2026-09-03 through the ticket pipeline (`/ticket-plan`). Full scope: `tickets/SEC-1/ticket.md`. Effort **L**; blast radius **high** (every frontend fetch needs the header). Pairs with [[SEC_2_Firestore_Rules]]: both are required, neither is sufficient, because the API's service account bypasses Firestore rules.

## What It Must Do

The ticket groups the exposure into four tiers, all reachable by an unauthenticated caller today:

| Tier | Routes | What a stranger gets |
|------|--------|----------------------|
| Role mutation | `POST /api/updateRole` | Grant any account `admin` |
| User enumeration | `GET /api/getUsers` with no `coachId` | Every user's uid, name, email, role |
| Single-athlete health | `/api/heart-rate`, `sleep`, `vo2max`, the compat aliases, `/getVO2MaxData` | Any athlete's time series by uid |
| Team health | `/api/team/*` | Several athletes at once, no roster check |

Fix: require `Authorization: Bearer <idToken>`, verify with `admin.auth().verifyIdToken()`, then authorize by role and roster membership. The ticket lists every client call site, including dead ones, so none is missed.

> The ticket was planned before Phase 1, which deleted the legacy `/updateRole` and `/getUsers` duplicates and changed `server/server.js` line numbers. Re-check its line references before building.

## Testing

- Not started.

## Resume Bullets

- *(write when built)*

## Follow-ups / Open Questions

- Decide whether it must land before the capstone team touches real athlete data (open question in [[Standup/README|Standup]]).

---

## Related Notes

- [[SEC_2_Firestore_Rules]]
- [[API_Endpoints]]
- [[Auth_And_Roles]]
- [[Known_Issues]]
