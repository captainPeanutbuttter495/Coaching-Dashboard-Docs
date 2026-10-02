# Configuration

Every environment variable across the three processes, where each is loaded from, and what breaks without it.

---

## Overview

Three processes, three separate configurations, three different loading mechanisms. The frontend's values are **baked in at build time** and are public; the API's are secret and read at boot; the email service reads a `.env` through `dotenv`.

Since refactor Phase 2 the frontend env file lives at the **repo root** (`.env.local`), not in `src/`. Older notes and the pre-refactor README say `src/.env.local`; that is no longer read.

---

## Frontend: `.env.local` (repo root)

Loaded by Vite. Copy `.env.example` to start.

| Variable | Purpose |
|----------|---------|
| `VITE_FIREBASE_API_KEY` | Firebase Web SDK config. All eight `VITE_FIREBASE_*` values come from Project settings → Your apps → Web app |
| `VITE_FIREBASE_AUTH_DOMAIN` | |
| `VITE_FIREBASE_DATABASE_URL` | |
| `VITE_FIREBASE_PROJECT_ID` | |
| `VITE_FIREBASE_STORAGE_BUCKET` | |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | |
| `VITE_FIREBASE_APP_ID` | |
| `VITE_FIREBASE_MEASUREMENT_ID` | |
| `VITE_API_BASE_URL` | Origin of the Insights API. **Empty locally** (the Vite proxy handles it); the Render URL for any deployed build |

Read in `src/lib/firebase.js` and `src/lib/apiBase.js`.

### Two things that bite

1. **A missing or misplaced env file fails as a Firebase error, not a config error.** Every `import.meta.env.VITE_*` is `undefined`, and Firebase throws `auth/invalid-api-key` at load. There is no warning that the file was not found.
2. **`VITE_*` values are baked into the bundle at build time.** Changing `VITE_API_BASE_URL` on Netlify requires a rebuild. And anything in a `VITE_` variable is **public**: it ships to every browser. That is fine for Firebase Web config, which is designed to be public; it is not fine for anything else.

---

## Insights API: process environment

| Variable | Required | Purpose |
|----------|----------|---------|
| `FIREBASE_SERVICE_ACCOUNT_JSON` | effectively yes | The entire service-account JSON as **one string**, parsed with `JSON.parse` |
| `FRONTEND_ORIGIN` | in production | Added to the CORS allowlist beside `http://localhost:5173` |
| `PORT` | no | Defaults to 3000 |

```js
if (process.env.FIREBASE_SERVICE_ACCOUNT_JSON) {
  serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT_JSON);
} else {
  serviceAccount = require("./firebase-service-account-prototype.json");
}
```

Locally you can skip the variable and drop the key file at `server/firebase-service-account-prototype.json`. `server/service-account.example.json` shows the shape. With neither present the server now exits with an error **naming both supply routes** (added in Phase 2), instead of a raw `MODULE_NOT_FOUND`. Malformed JSON in the variable throws at boot. The server never starts half-configured.

**The service account and the web config must belong to the same Firebase project** (`coach-student-invite`). A key for the older `smart-clothing-app` project loads fine and then fails every roster request as "Coach not found", because the uids belong to a different Auth tenant.

**Never commit a service account.** It is a fully privileged credential for the whole project, and the API does no authorization on top of it ([[API_Endpoints]]).

---

## Email Service: `email-service/.env`

Loaded with `require("dotenv").config()`.

| Variable | Purpose |
|----------|---------|
| `EMAIL_HOST` | SMTP host: `localhost` for MailHog |
| `EMAIL_PORT` | SMTP port: `1025` for MailHog |

There is no `PORT` variable; the listen port is hard-coded to 3001. The transport is hard-coded to `secure: false, auth: null`, so these two variables cannot point the service at a real authenticated SMTP provider without a code change. See [[Email_Service]].

---

## Where Each Is Set in Production

| Piece | Where |
|-------|-------|
| Frontend `VITE_*` | Netlify → Site settings → Environment variables, applied at build |
| API `FIREBASE_SERVICE_ACCOUNT_JSON`, `FRONTEND_ORIGIN` | Render → Environment |
| Email service | not deployed |

---

## Key Files

| File | Purpose |
|------|---------|
| `.env.example` | Frontend template (committed) |
| `.env.local` | Frontend values (gitignored) |
| `server/service-account.example.json` | Credential shape (committed) |
| `server/firebase-service-account-prototype.json` | Local API credential (gitignored) |
| `email-service/.env` | Email service (gitignored via `.env`) |

`.gitignore` covers `.env`, `.env.local`, `.env.*.local`, the legacy `src/.env.local`, and both service-account paths.

---

## Related Notes

- [[Local_Dev_Setup]]
- [[Deployment]]
- [[API_Endpoints]]
- [[03-Container]]
