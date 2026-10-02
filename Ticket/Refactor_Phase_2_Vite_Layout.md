---
title: "Refactor Phase 2: Normalize the Vite Layout"
date_started: 2026-09-05
date_finished: 2026-09-05
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: 45f4258
status: committed
standups: []
skills: [vite, build-tooling, netlify, firebase]
tags: [ticket]
---

# Refactor Phase 2: Normalize the Vite Layout

Dropped `root: "src"` from the Vite config and fixed the deploy bugs that one line had been causing: a missing SPA fallback, a case-sensitivity image duplicate, and Firebase initializing only by accident.

---

## Ticket

Refactor Phase 2. See [[Refactor]] problem 3 ("the build fought the framework's conventions").

## What I Did

- `vite.config.js`: removed `root`; added the `@` → `src` alias and `build.outDir: "dist"`; fixed `test.setupFiles`; dropped a `dotenv` import that read a nonexistent file.
- Moved `src/index.html` → `index.html` and removed a duplicate stylesheet `<link>` that served unprocessed `@tailwind` directives.
- Split the entry from the router: new `src/main.jsx` mounts and **explicitly imports `./lib/firebase`** before rendering; `App.jsx` is the router only. This closed the "Firebase initializes as an import side effect" bug that 17 bare `getAuth()` / `getFirestore()` call sites depended on.
- `firebaseConfig.js` → `src/lib/firebase.js`; `src/Public/ARCS.jpg` → `src/assets/images/`.
- `netlify.toml`: publish `dist`; SPA fallback moved into `[[redirects]]`; deleted `public/_redirects`, which had never reached the build.
- Added `jsconfig.json`, `.env.example`, `server/service-account.example.json`, and a credential error in `server/server.js` that names both supply routes.
- Moved 108 KB of sample CSVs from `public/data/` to `docs/sample-data/` so they would not start being published.

## Technical Details

| File | Change |
|------|--------|
| `vite.config.js` | no `root`, `@` alias, `outDir: dist` |
| `index.html` | moved to repo root |
| `src/main.jsx` | new: mount + explicit Firebase import |
| `src/lib/firebase.js` | moved from repo-root `firebaseConfig.js` |
| `netlify.toml` | `publish = "dist"`, `[[redirects]]` |
| `.env.example`, `server/service-account.example.json` | new committed templates |

Files were moved with plain `mv`, not `git mv`, because staging trips the git-guard hook; git detects the renames by content.

## Testing

- `npm run build` green, 2,539 modules, output at repo-root `dist/`; no stray `dist/ARCS.jpg`.
- Dev server serves `/`, `/src/main.jsx` and the aliased image; `/coach` falls back to the SPA shell.
- Not verified: the `[[redirects]]` block on a real Netlify deploy.

## Impact

Fixed a deep-link 404 on the deployed site, a Windows-only build difference, and an initialization order that any tidy-up could have broken.

## Resume Bullets

- **Diagnosed and fixed three production deploy bugs** traced to one non-standard Vite setting: a missing SPA fallback on Netlify, a case-sensitivity build difference between Windows and Linux, and implicit Firebase initialization.
- **Made app initialization deterministic** by splitting the React entry point from the router and importing Firebase explicitly before render.

## Follow-ups / Open Questions

- Owner step at the time: move `src/.env.local` to the repo root.

---

## Related Notes

- [[Refactor]]
- [[Deployment]]
- [[Configuration]]
- [[Refactor_Phase_3_Features]]
