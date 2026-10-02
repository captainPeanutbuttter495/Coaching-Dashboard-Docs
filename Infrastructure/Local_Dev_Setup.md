# Local Dev Setup

How to run the whole system on one machine: install, configure, start three processes, and give yourself a role.

---

## Overview

Three processes. Two of them are required for a working app: the Vite dev server and the Insights API. The email service starts but nothing calls it today. `npm run dev:all` runs all three in one terminal.

---

## Prerequisites

- Node.js 18+ and npm
- A Firebase project with Authentication and Cloud Firestore (the team's is `coach-student-invite`)
- A service-account JSON for **that same project** (Project settings → Service accounts → Generate new private key)
- Optional: MailHog on `localhost:1025` to catch invitation email

## 1. Install

```bash
npm install
```

`nodemailer` is declared since refactor Phase 1, so no extra install is needed.

## 2. Environment

Full reference: [[Configuration]]. The short version:

- Copy `.env.example` to **`.env.local` at the repo root** and fill in the eight `VITE_FIREBASE_*` values.
- Leave `VITE_API_BASE_URL` **empty** so the Vite proxy handles `/api/*`.
- For the API, set `FIREBASE_SERVICE_ACCOUNT_JSON` or drop the key file at `server/firebase-service-account-prototype.json`. Both are gitignored.

## 3. Run

| # | Command | Port | What breaks without it |
|---|---------|------|------------------------|
| 1 | `npm run dev` | 5173 | everything |
| 2 | `npm run dev:api` (`node server/server.js`) | 3000 | every chart; login still works |
| 3 | `npm run dev:email` (`node email-service/server.js`) | 3001 | nothing today (no caller) |

Or **`npm run dev:all`**: `concurrently -k` runs all three with `[web]` / `[api]` / `[email]` prefixes and **stops the others when one exits**. An API that cannot start (missing service account, port in use) ends the session with a visible error instead of leaving a UI whose every request 502s. Added in Phase 6. Note that `-k` also kills Vite and the API if the *email* service dies.

Terminal 1 alone gives you a UI whose `/api/*` calls all fail. Login works, charts do not: that asymmetry is the signature of a missing API process. See [[03-Container]].

Expected startup lines:

```
VITE ready in NNN ms  ➜  Local: http://localhost:5173/
Server listening on port 3000
Email service running at http://localhost:3001
```

## 4. Get Yourself a Role

Signing in is not enough. `ProtectedRoute` reads `Users/{uid}.role`, and an Auth account without that document sits on `Loading role...`. `Register.jsx` creates the document with `role: "coach"`; for any other role, or an account created elsewhere, edit it in the Firebase console:

```
Users/{your-uid}
  role: "coach"
```

To see anything on [[Team_Insights]] your coach document also needs `authorizedAthletes`: an array of **DocumentReferences** to other `Users/*` documents, not uid strings. See [[Data_Model]].

## 5. Pick a Date Range That Has Data

The newest health sample in the project is **2025-03-27**. Every picker defaults to a recent window, so charts come up blank. Choose **2025-03-26 → 2025-03-28**.

---

## Troubleshooting

| Symptom | Cause |
|---------|-------|
| Firebase `auth/invalid-api-key` on load | no `.env.local` at the repo root (an old `src/.env.local` is ignored) |
| Charts empty, no console error | API not running, the date range has no data, or the athlete has no documents |
| Charts empty, `502` in the network tab | API not running |
| Athlete list empty, `8 RESOURCE_EXHAUSTED` from the API | Firestore daily read quota spent; resets daily. See [[API_Endpoints]] |
| "Coach not found" for a coach that exists | service account is for a different Firebase project than the web config |
| `/api/getHeartRate` 404s on the server | someone re-added the proxy `rewrite` that strips `/api` |
| "You are not authorized to access this page." after a refresh | permission context is only set at login. See [[Auth_And_Roles]] |
| Blank page right after login | no recognized `role`: `Login.jsx` navigates to the nonexistent `/home` |
| API exits at boot naming two credential options | no service account supplied |
| Fewer chart points than expected | rows silently dropped by `toDate()`. See [[Data_Model]] |

---

## Other Commands

| Command | Notes |
|---------|-------|
| `npm run build` | → `dist/` at the repo root |
| `npm run preview` | Serve the built output |
| `npm run lint` / `lint:fix` | ESLint over `src`, `server`, `email-service` and root configs. **0 errors expected**; the 39 warnings are a known backlog item, visible on purpose. Enforces the layering rule and case-sensitive imports ([[Project_Structure]]) |
| `npm run format` / `format:check` | Prettier over `src`, `server`, `email-service` |
| `npm test` / `npm run test:run` | Vitest; hits live Firebase. See [[Testing]] |

---

## Key Files

| File | Purpose |
|------|---------|
| `package.json` | Scripts |
| `vite.config.js` | Dev proxy to :3000 |
| `.env.example` | Frontend env template |
| `server/service-account.example.json` | Credential shape |

---

## Related Notes

- [[Configuration]]
- [[Deployment]]
- [[Known_Issues]]
- [[Tech_Stack]]
