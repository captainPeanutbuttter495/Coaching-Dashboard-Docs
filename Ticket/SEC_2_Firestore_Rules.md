---
title: "SEC-2: Scope Firestore Rules to the Data Each Caller Owns"
date_started: 2026-09-03
date_finished:
repo: coaching-dashboard
area: api
branch:
pr:
commit:
status: planned
standups: []
skills: [security, firestore-rules]
tags: [ticket]
---

# SEC-2: Scope Firestore Rules to the Data Each Caller Owns

*(planned)* Replace the "any signed-in user can read and write anything" rules with rules scoped to what each caller owns, and make them deployable and testable from the repo.

---

## Ticket

Planned 2026-09-03 through the ticket pipeline. Full scope: `tickets/SEC-2/ticket.md`. Effort **M**; blast radius **high** (a too-tight rule locks out the app). Pairs with [[SEC_1_API_Auth_Tokens]].

## What It Must Do

- Today `firestore.rules` ends with `match /{document=**} { allow read, write: if request.auth != null; }`. Registration is open, so "any signed-in user" means anyone. That includes writing every athlete's health subcollections and **promoting yourself to `admin`** from the browser console.
- Delete the leftover date-limited block from the SmartClothingApp (expired 2025-08-24).
- **Precondition:** there is no `firebase.json` or `.firebaserc`, so nothing in the repo deploys, validates or emulator-tests the rules, and the live rules may not be this file.
- Health subcollections are never client-writable, by any role; writes happen server-side with `firebase-admin`.

## Testing

- Not started. Needs the Firestore emulator, which needs `firebase.json`.

## Resume Bullets

- *(write when built)*

## Follow-ups / Open Questions

- Confirm what the live project's rules actually are before changing this file.

---

## Related Notes

- [[SEC_1_API_Auth_Tokens]]
- [[Data_Model]]
- [[Auth_And_Roles]]
- [[Known_Issues]]
