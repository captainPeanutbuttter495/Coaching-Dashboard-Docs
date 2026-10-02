# Tech Stack

Every technology in the repo, grouped by the container that uses it, with the versions pinned in `package.json` on `refactor @ 99a9eb6`.

---

## Overview

One repository, one `package.json`, one `node_modules`, three runnable processes: a React SPA, an Express API, and an Express email service. There is no shared code module between the frontend and either server. See [[03-Container]].

The toolchain is **end-of-life across the board** (ESLint 8, Prettier 2, Vite 3, Vitest 0.24). Upgrading it is a backlog ticket; Phase 6 deliberately changed no version. See [[Roadmap]].

---

## Frontend: Web SPA

| Technology | Version | Used for |
|------------|---------|----------|
| React | ^18.2.0 | UI |
| React Router DOM | ^6.22.3 | Routing (`src/App.jsx`) |
| Vite | ^3.1.4 | Dev server (:5173), build to `dist/` |
| `@vitejs/plugin-react` | ^2.1.0 | JSX transform (automatic runtime, so no `import React`) |
| Tailwind CSS | ^3.4.1 | **All** styling, unprefixed. `src/style.css` is three `@tailwind` lines. See [[Styling_And_Charts]] |
| PostCSS + Autoprefixer | ^10.4.17 | Tailwind pipeline |
| Firebase Web SDK | ^10.11.0 | Auth, role lookup, invitations (`src/lib/firebase.js`) |
| `react-firebase-hooks` | ^5.1.1 | `useAuthState` in `ProtectedRoute` |
| `react-permission-role` | ^2.0.0 | `<AllowedAccess>`, the second role gate. See [[Auth_And_Roles]] |
| Recharts | ^3.1.2 | Newer coach-dashboard cards, `SleepTimeline` |
| Chart.js + `react-chartjs-2` | ^4.4.3 / ^5.2.0 | Older per-athlete and team-insights charts |
| `chartjs-adapter-date-fns`, `-plugin-annotation`, `-plugin-zoom` | ^3.0.0 / ^3.1.0 / ^2.2.0 | Chart.js time axes, annotations, zoom |
| `date-fns` | ^4.1.0 | Required peer of the Chart.js date adapter |
| `moment` | ^2.30.1 | Used by `SleepTimeline.jsx` |
| MUI (`@mui/material`, `@mui/x-data-grid`) + Emotion | ^5.15.21 / ^7.7.1 | `Admin`, `Invitation`, `Login`, `Register` |
| FontAwesome, `react-icons` | ^6.x / ^5.0.1 | Icons |
| `axios` | ^1.6.8 | One call, in `Vo2MaxChart.jsx` |

## API: Insights API

| Technology | Version | Used for |
|------------|---------|----------|
| Node.js | 18+ | Runtime |
| Express | ^4.19.2 | `server/server.js`, 12 routes. See [[API_Endpoints]] |
| `firebase-admin` | ^12.1.0 | Service-account access to Firestore and Auth |
| `cors` | ^2.8.5 | Allowlist: `localhost:5173` + `FRONTEND_ORIGIN` |

## Email service

| Technology | Version | Used for |
|------------|---------|----------|
| Express | ^4.19.2 | `email-service/server.js`, one route |
| `nodemailer` | ^10.0.0 | SMTP (declared since Phase 1) |
| `dotenv` | ^16.4.5 | Loads `EMAIL_HOST` / `EMAIL_PORT` |

## Data and hosting

| Service | Role |
|---------|------|
| Cloud Firestore | System of record. Written by the smart-clothing mobile app, read here. See [[Data_Model]] |
| Firebase Authentication | Sign-in; display names for the API |
| Netlify | Hosts the SPA (`netlify.toml`) |
| Render | Hosts the API |
| MailHog | Local SMTP catcher for the email service |

## Tooling

| Tool | Version | Notes |
|------|---------|-------|
| ESLint | 8.24.0 | Legacy `.eslintrc.cjs`. `react`, `react-hooks`, `jsx-a11y`, `import` plugins; `prettier` last |
| `eslint-import-resolver-alias` | ^1.1.2 | Makes `@/…` resolvable, which is what lets the layering rule work |
| Prettier | 2.7.1 | `.prettierrc` holds explicit options |
| Vitest + happy-dom | ^0.24.3 / ^17.5.6 | Tests. Config lives in `vite.config.js` |
| `@testing-library/react` | ^13.4.0 | Component tests |
| `vitest-fetch-mock` | ^0.2.1 | Installed, unused |
| `concurrently` | ^10.0.5 | `npm run dev:all` |

---

## Key Files

| File | Purpose |
|------|---------|
| `package.json` | Every dependency and script |
| `vite.config.js` | `@` alias, `dist/` output, dev proxy, vitest config |
| `tailwind.config.js` | Content globs. No `screens`, no prefix |
| `.eslintrc.cjs` | Lint rules, including the enforced layering rule |
| `netlify.toml` | Build command, publish dir, SPA fallback |

---

## Related Notes

- [[Project_Structure]]
- [[03-Container]]
- [[Styling_And_Charts]]
- [[Local_Dev_Setup]]
