# API Endpoints

Every route of the Insights API: roster, per-athlete time series, team time series, and one legacy path. All in `server/server.js`.

---

## Overview

One Express file, 655 lines, no router modules, **12 routes**. Phase 1 deleted the duplicated legacy `/getUsers` and `/updateRole` routes and the commented-out earlier variants. What the file still has is the shape: every route does a full Firestore subcollection read and filters in Node, and there is **no authentication middleware**.

Structure: [[05-Component-API]]. The Firestore shape these routes read: [[Data_Model]].

---

## Middleware

```js
const allowedOrigins = ["http://localhost:5173", process.env.FRONTEND_ORIGIN].filter(Boolean);
```

CORS rejects anything else with `CORS blocked origin: …`. A request with **no** `Origin` header (curl, Postman, another server) is allowed through; the callback treats `!origin` as permitted. `express.json()` is the only other middleware.

---

## Endpoints

| Method | Path | Params | Returns |
|--------|------|--------|---------|
| GET | `/api/getUsers` | `?coachId=` | `{ success, users: [{ id, firstName, lastName, email, sports, role }] }`: the coach's authorized athletes |
| GET | `/api/getUsers` | *no `coachId`* | same shape, **every user in the project** (the Admin roster) |
| POST | `/api/updateRole` | body `{ email, newRole }` | `{ success, message }`. **No caller in the UI** since the Admin form was removed |
| GET | `/api/heart-rate` | `?userId&start&end` | `[{ timestamp, bpm }]` |
| GET | `/api/sleep` | `?userId&start&end` | `[{ start, end, stage }]` |
| GET | `/api/vo2max` | `?userId&start&end` | `[{ timestamp, vo2max }]` |
| GET | `/api/getHeartRate` | `?userId&start&end` | `{ heartRateData: [{ time, beatsPerMinute }] }` (compat alias) |
| GET | `/api/getSleepData` | `?userId&start&end` | `{ sleepData: [{ startTime, endTime, stage }] }` (compat alias) |
| GET | `/getVO2MaxData` | `?userId&start&end` | `{ data: [{ x, y }] }` (legacy, unprefixed) |
| GET | `/api/team/heart-rate` | see below | `{ series }` or `{ average }` |
| GET | `/api/team/sleep` | see below | `{ series }` or `{ average }` |
| GET | `/api/team/vo2max` | see below | `{ series }` or `{ average }` |
| GET | `/api/team/respiratory-rate` | see below | `{ series }` or `{ average }` |

Missing `userId` on a per-athlete route → **400** `{ message: "Missing userId" }`. `start` / `end` are optional; omitting them returns everything.

### Roster

The `coachId` branch resolves `authorizedAthletes` DocumentReferences one at a time and joins each with `admin.auth().getUser()`. An athlete whose Auth record is missing is skipped with a `console.warn`, not an error, so a roster can silently come back short. An unknown coach returns `{ success: false, message: "Coach not found" }`.

### Per-athlete compat aliases

`/api/getHeartRate` and `/api/getSleepData` return the same data as `/api/heart-rate` and `/api/sleep`, wrapped in an object with the field names the older components expect. `AthleteInfo.jsx` uses them. `/getVO2MaxData` is the one route never given an `/api` prefix; `Vo2MaxChart.jsx` calls it, and the Vite proxy has a dedicated entry for it. New code should use the plain `/api/*` routes. There is no per-athlete respiratory-rate route.

---

## Team Time Series

One shared contract across four metrics:

```
GET /api/team/{heart-rate,sleep,vo2max,respiratory-rate}
      ?athleteIds=uid1,uid2,uid3      required, comma-separated; missing → 400
      &start=<ISO>&end=<ISO>
      &mode=overlay|grid|avg          default overlay

overlay / grid  →  { "series": [ { "userId", "label", "data": [ … ] } ] }
avg             →  { "average": [ … ] }
```

`overlay` and `grid` return identical payloads; the UI decides how to draw them. [[Team_Insights]] only ever sends `overlay` or `avg`.

### The heart-rate divergence

Three of the four routes share `buildTeamSeries()`. `heart-rate` does not; it predates the helper and kept its own implementation. `AGENTS.md` says all four share it, which is not quite true:

| | `sleep`, `vo2max`, `respiratory-rate` | `heart-rate` |
|---|---|---|
| Fetch helper | `buildTeamSeries()` | `fetchAthleteHeartRows()` |
| Athlete fetches | **serial** `for` loop | **parallel** `Promise.all` |
| Label lookup | `getAthleteLabel()`: Auth `displayName` → Firestore name → uid | inline: `displayName` → **email** → uid |
| Point value key | `value` (and `bpm` mirrored) | `bpm` only |
| `mode=avg` bucketing | **exact matching timestamp string** | **60-second bins**, rounded to 1 decimal |
| Averaged point shape | `{ timestamp, value }` | `{ timestamp, bpm }` |

The averaging difference matters most. `averageSeries()` groups by the exact ISO timestamp string, so two athletes only contribute to the same average if their samples landed on the *identical* instant. For sparse daily metrics like VO₂ max that mostly works; for anything sampled continuously the "average" is usually a set of single-athlete points. The heart-rate route's minute binning is the more correct approach and is the one to generalize.

### `buildTeamSeries` output quirk

It emits every possible key per point, whether or not the metric has one:

```js
data: rows.map((row) => ({
  timestamp: row.timestamp,
  start: row.start,      // sleep only; undefined otherwise
  end: row.end,          // sleep only
  stage: row.stage,      // sleep only
  value: row.value ?? row.bpm,
  bpm: row.value ?? row.bpm,
}))
```

So a VO₂ max point carries `start: undefined`, and a sleep point carries `bpm` equal to its stage number. Harmless today; confusing to read.

### Respiratory rate returns nothing

Probed live on 2026-09-05 with a 2020–2026 window: heart-rate 529 points, sleep 28, VO₂ max 12, respiratory-rate **0** for the same athlete. Either `RespiratoryRateDataHX` is empty or `pickRows` does not match the stored field spelling. Unresolved.

---

## Performance Shape

Every route does a **full subcollection `.get()`** and filters in Node afterwards (`clampAndSort`). `.where()`, `orderBy` and `limit` appear nowhere in the file. Cost scales with total documents stored, not with the window requested.

**This is no longer theoretical.** On 2026-09-07 it exhausted the Spark plan's 50,000 reads/day (queries 50,148 against `/Users` 152) and took every data page down with `8 RESOURCE_EXHAUSTED`. One `/api/team/heart-rate` for an 18-athlete roster is ≈ 18 × 530 ≈ 9,500 reads, so five or six ordinary interactions spend the whole day's budget. Reproduce:

```bash
curl "localhost:3000/api/getUsers?coachId=<uid>"
# {"success":false,"message":"8 RESOURCE_EXHAUSTED: Quota exceeded."}
```

The real fix is a Firestore range query on the timestamp field, which the inconsistent field naming ([[Data_Model]]) makes awkward; a canonical-timestamp backfill unlocks it. Cheaper interim: query per known field name and merge, or `.limit()` plus a short server cache. Client-side caching in Team Insights cuts the request count and needs no migration.

---

## No Authentication

**The API verifies no Firebase ID token.** Any caller who can reach the host can request any athlete's data by uid, call `/api/getUsers` with no `coachId` to enumerate every user, or `POST /api/updateRole` to make themselves an admin. The server's service account is fully privileged, and the Admin SDK bypasses Firestore rules.

The planned fix, [[SEC_1_API_Auth_Tokens]]: require `Authorization: Bearer <idToken>`, verify with `admin.auth().verifyIdToken()`, then check the caller's role and roster membership. Every one of the 12 client call sites needs the header.

---

## Key Files

| File | Purpose |
|------|---------|
| `server/server.js` | Every route and helper |
| `server/service-account.example.json` | Shape of the credential the server loads |
| `vite.config.js` | Dev proxy for `/api` and `/getVO2MaxData` |

## Tests

None. `toDate`, `clampAndSort`, `pickRows`, `buildTeamSeries` and `averageSeries` are unexported, which blocks testing them. See [[Testing]].

---

## Related Notes

- [[05-Component-API]]
- [[Data_Model]]
- [[Auth_And_Roles]]
- [[Team_Insights]]
- [[Configuration]]
- [[Known_Issues]]
