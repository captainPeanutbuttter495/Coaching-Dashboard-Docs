# Known Issues

Tech debt, defects and risks on `refactor @ 99a9eb6`. Every item was checked against the code; items marked **live** were observed failing, not just inferred. The candidate-ticket version of this list, with effort and blast radius, is `tickets/BACKLOG.md`.

---

## Overview

The system currently assumes everyone who can reach it is trusted. Items 1–3 are a posture decision that must be made before the app holds real athlete data. Item 4 is actively failing. Most of the rest are silent: an empty chart rather than an error.

---

## Security

| # | Issue | Where | Effect |
|---|-------|-------|--------|
| 1 | **The API verifies no auth token on any route** | `server/server.js` (all 12 routes) | Anyone who can reach the host can read any athlete's data by uid, enumerate every user with email via `/api/getUsers` with no `coachId`, or `POST /api/updateRole` to grant themselves admin. CORS restricts browsers, not clients; a request with no `Origin` is allowed by design. Planned fix: [[SEC_1_API_Auth_Tokens]] |
| 2 | **`firestore.rules` lets any signed-in user read and write any document** | `firestore.rules` | Every athlete's health subcollections, and every user's own `role` field, are writable from the browser console. Also holds an expired 2025-08-24 catch-all from an earlier app. There is no `firebase.json`, so nothing in the repo deploys or tests these rules. Planned fix: [[SEC_2_Firestore_Rules]] |
| 3 | **Login logs the full user document to the console** | `src/pages/Login.jsx:53-55`, `src/components/global/ProtectedRoute.jsx:21` | Email, roster, invitations and role land in every real user's browser console, in production |

## Data / Correctness

| # | Issue | Where | Effect |
|---|-------|-------|--------|
| 4 | **live** · Unbounded reads exhausted the Firestore daily quota | every route: full `.get()`, no `.where()` | On 2026-09-07 the Spark plan's 50,000 reads/day ran out (queries 50,148, `/Users` 152) and every data page went empty with `8 RESOURCE_EXHAUSTED`. One team heart-rate request for 18 athletes costs ≈ 9,500 reads. See [[API_Endpoints]] |
| 5 | API failures render as empty data | `src/pages/TeamInsights.jsx:79-81`, `AthleteInfo.jsx` | A 500, a dead API and an empty roster look identical. Item 4 was first mistaken for a CSS regression because of this |
| 6 | Only one page works against the deployed API | `Admin.jsx`, `AthleteInfo.jsx`, `Vo2MaxChart.jsx` use bare paths | Only `TeamInsights` uses `apiUrl()`. The rest resolve only through the Vite dev proxy, so deployed they are empty. See [[Frontend_Data_Flow]] |
| 7 | Permission context does not survive a refresh | `react-permission-role`, set only in `Login.jsx` | After a hard refresh `<AllowedAccess>` shows "You are not authorized" on `CoachDashboard`, `Admin`, `AthleteAccess` and all of `settings/*`. See [[Auth_And_Roles]] |
| 8 | `Login.jsx` navigates to `/home`, which is not a route | `src/pages/Login.jsx:98,102,106` | A user with a missing or unknown role gets a blank page |
| 9 | Nothing in the repo closes the invitation → roster loop | `src/features/invitations/api/sendInvitation.js` | Invitations write `pendingPermissions` / `invitationsSent`; nothing writes `authorizedAthletes` or `coachList`. Existing coaches have rosters (written outside this repo); **new** coaches start empty with no error. See [[Data_Model]] |
| 10 | The invitation email is never sent | `sendInvitation.js` | The `POST :3001/send-invitation` call has been commented out since before the refactor, so [[Email_Service]] has no caller |
| 11 | All health data is stale | Firestore | Newest sample anywhere is 2025-03-27. Date pickers default to a recent window, so charts look broken. Use 2025-03-26 → 2025-03-28 to see data |
| 12 | `/api/team/respiratory-rate` returns 0 points | `server/server.js` | Same athlete returns 529 heart-rate points. Either the subcollection is empty or `pickRows` misses the stored field name |
| 13 | Date windows are UTC, not local | `src/lib/dates.js` | A Pacific coach asking for "September 3" gets 5pm Sep 2 – 5pm Sep 3 local. Wrong for sleep |
| 14 | Team heart-rate diverges from the other three team routes | `server/server.js` | Different helper, parallel fetch, `bpm` key, 60s-bin averaging. The others' exact-timestamp averaging returns single-athlete points labelled as an average. See [[API_Endpoints]] |
| 15 | Rows with unparseable timestamps are dropped silently | `clampAndSort` | Fewer chart points, no log |
| 16 | `Register.jsx` hard-codes `role: "coach"` | `src/pages/Register.jsx` | The form can only create coaches; also writes `institution` / `teamName`, which nothing reads |

## UI / Coach Dashboard

| # | Issue | Where | Effect |
|---|-------|-------|--------|
| 17 | The card grid never collapses and overflows below ~900px | `src/pages/CoachDashboard.jsx` | Faithfully preserved by the refactor; needs a narrow-screen design. Remove `[container-type:inline-size]` at the same time. See [[Coach_Dashboard]] |
| 18 | Team-builder roster tabs filter on `athlete.group`, which no row has | `MiniTeamBuilderCard.jsx` | Every tab but "All Athletes" is empty |
| 19 | Two divergent 4-3-3 coordinate sets | `MiniTeamBuilderCard.jsx` | Module-scope `formationSlots` vs. one built in `addPlayer()` |
| 20 | Frozen strings that look derived | `data/loadZones.js`, `data/vo2Trend.js` | Slide values, "+2 last 7 days" badge; flagged in the data modules |
| 21 | KPI row says 12 athletes; builder roster and Athlete Access hold 18 | `data/kpis.js` | Inconsistent mock data |
| 22 | "Andrew Rivers" hardcoded a third time | `features/settings/components/SettingsNavbar.jsx` | Should come from an auth-backed profile |
| 23 | Inert buttons and ignored props | `AthleteAccess.jsx` (Cancel/Revoke/Save have no `onClick`), `PendingInvitations` (ignores its prop), `Admin.jsx` (`_message` never shown) | Invitations data flows to nothing |
| 24 | Two components inject global `<style>` blocks at runtime | `PopupCalendar.jsx`, `SyncedSegmentedToggle.jsx` | `.sidebar` / `.day` are generic enough to collide; `PopupCalendar` also shows no month label |
| 25 | `.new-team-btn`'s declared padding/size/weight never applied | `MiniTeamBuilderCard.jsx` | Preserved as rendered; matching intent is a visual decision |

## Performance

| # | Issue | Where | Effect |
|---|-------|-------|--------|
| 26 | Roster and team reads are N+1 and serial | `/api/getUsers`, `buildTeamSeries` | `/api/team/heart-rate` already uses `Promise.all`; follow it |
| 27 | One ~2.29 MB JS chunk, no code splitting | `vite.config.js`, `App.jsx` | Every page and chart library loads before login paints. `React.lazy` the routes |
| 28 | No client-side caching | every page | Team Insights refetches every athlete's full history on each toggle, which feeds item 4 |

## Tooling / Structure

| # | Issue | Where |
|---|-------|-------|
| 29 | 39 lint warnings (37 `jsx-a11y`, 2 `exhaustive-deps`), left visible on purpose | 10 files; see `tickets/BACKLOG.md` |
| 30 | Toolchain is end-of-life (ESLint 8, Prettier 2, Vite 3, Vitest 0.24) | `package.json` |
| 31 | Tests need live Firebase; two test credentials are broken | `src/__tests__/` · [[Testing]] |
| 32 | `server/server.js` is one 655-line file; helpers unexported, which blocks testing them | `server/server.js` |
| 33 | ~35 inline `style={{}}` stragglers outside the coach dashboard | `SleepTimeline`, `PopupCalendar`, `Admin`, `Login`, `Register`, settings pages |

---

## Resolved by the Refactor

Kept so a reader of an older note or the 2026-09-03 README knows these are fixed.

| Was | Fixed in |
|-----|----------|
| `nodemailer` not in `package.json` | [[Refactor_Phase_1_Dead_Code]] |
| `apexcharts`, `react-apexcharts`, `@tanstack/react-query`, Bootstrap, daisyUI and ~15 other unused packages | [[Refactor_Phase_1_Dead_Code]] |
| `CoachDashboard2.jsx` and 9 other orphan files; legacy `/getUsers`, `/updateRole` routes | [[Refactor_Phase_1_Dead_Code]] |
| `public/_redirects` never reached the build (SPA fallback missing) | [[Refactor_Phase_2_Vite_Layout]]: now `[[redirects]]` in `netlify.toml` |
| `.env.local` had to live in `src/`; build went to `src/dist` | [[Refactor_Phase_2_Vite_Layout]] |
| Firebase initialized only as an import side effect | [[Refactor_Phase_2_Vite_Layout]]: `src/main.jsx` imports `src/lib/firebase.js` |
| `src/Public` vs `src/public` case bug duplicated images in the build | [[Refactor_Phase_2_Vite_Layout]] |
| `npm run lint` could not run (no config) | [[Refactor_Phase_6_Tooling]] |
| 3,398-line hand-written stylesheet and the `tw-` prefix | [[Refactor_Phase_5_Styling_Pass]] → [[Refactor_Phase_8C_Team_Builder]] |

---

## Related Notes

- [[Roadmap]]
- [[API_Endpoints]]
- [[Auth_And_Roles]]
- [[Testing]]
- [[Deployment]]
