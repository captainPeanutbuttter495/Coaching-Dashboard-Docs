# Testing

There is a test runner, two spec files, five tests, and no meaningful coverage. The refactor's safety net was browser measurement instead.

---

## Overview

`npx vitest run` passes **5 tests in 2 files** on `refactor @ 99a9eb6`, but the tests are live-Firebase smoke tests, not unit tests, and one passes only because it swallows its own failure. The refactor phases were verified by **build module counts, bundle diffs, and computed-style dumps in a real browser**, recorded in `docs/refactor-phases.md`. That method is described below because it is the closest thing this project has to a regression suite.

---

## What Exists

Configuration lives inside `vite.config.js`, not a separate `vitest.config.js`:

```js
test: {
  environment: "happy-dom",
  setupFiles: ["./src/setupVitest.js"],
}
```

```bash
npm test                                       # vitest, watch mode
npm run test:run                               # single pass
npx vitest run src/__tests__/login.test.jsx    # one file
npx vitest run -t "can login"                  # one test by name
```

### They are not unit tests

**The specs sign in against live Firebase with hardcoded credentials.** They need network access, a valid `.env.local` at the repo root, and the test accounts to still exist with the expected roles. So they fail on a fresh clone, offline, and in CI without secrets.

Two credentials are known to be broken:

- `routes.test.jsx`'s `dustin@gmail.com` returns `auth/invalid-credential`. The test passes because it catches the error.
- `login.test.jsx`'s `matthew@gmail.com` authenticates, but has **no `Users/{uid}` document**, so in the real app `ProtectedRoute` shows "Loading role..." forever.

`vitest-fetch-mock` is in `devDependencies`, which suggests someone intended to mock. It is not used.

---

## What Is Untested

Effectively everything that matters:

- the API: `server/server.js` has no tests at all, including `toDate()` and `clampAndSort()`, which are pure functions doing the trickiest work in the project ([[Data_Model]])
- `buildTeamSeries` / `averageSeries` and the heart-rate divergence ([[API_Endpoints]])
- `useDashboardAutoCycle`'s tick and tease state machine ([[Coach_Dashboard]])
- `apiUrl()` and the date helpers in `src/lib/`: small, pure, trivial to cover
- every chart component

## Where Testing Should Start

The highest value per line of test is the API's pure helpers. They are pure, they are where the real complexity lives, and they need no Firebase:

1. `toDate()`: the four accepted input types plus the null cases
2. `clampAndSort()`: range filtering, sorting, dropping unparseable rows
3. each metric's `pickRows`: every documented field-name spelling from [[Data_Model]]
4. `averageSeries()` vs. the heart-rate 60-second binning: pin the current behaviour before unifying

They are unexported module-scope functions in `server/server.js`, so step zero is exporting them. That is also the natural moment to split the file. Second priority: replace the live-Firebase login tests with mocked ones so the suite runs on a fresh clone.

---

## How the Refactor Was Verified Instead

Every refactor phase had the same acceptance criterion: *the running app looks and behaves exactly as before.* The evidence, in increasing strength:

| Check | Proves | Used in |
|-------|--------|---------|
| `npm run build` module count | Nothing was added or lost from the module graph | every phase |
| Bundle diff with minified identifiers normalized | The shipped JavaScript is the same | 6 |
| `renderToStaticMarkup` before/after | A component emits identical HTML | 4 |
| Full-page `getComputedStyle` dumps, ~100 properties per element, keyed by structural path, at 1920/1440/900/800/768px | Every element computes the same style | 5, 8A–8D |
| Noise floor measured before each edit | The comparison itself is stable (0 diffs on identical code) | 8A–8D |
| Forced pseudo-states via CDP `CSS.forcePseudoState` | Hover / active / disabled styles match the deleted rules | 8B, 8C |
| Region pixel diffs, per-region noise floor first | Paint is identical where screenshots are trustworthy | 8B–8D |

**Trust the dumps, not whole-page screenshots.** Phase 8A measured 30 of 105 screenshots differing between two runs of *identical code*, because Recharts and Chart.js rasterize differently run to run. The dumps over the same captures had a noise floor of zero. See [[Styling_And_Charts]].

The capture harness lived in session scratchpads and is **not in the repo**. Whether the capstone team inherits it is an open decision.

---

## Tests

| File | Tests | Description |
|------|-------|-------------|
| `src/__tests__/login.test.jsx` | 4 | Renders; login with valid, wrong-password and wrong-email credentials (live Firebase) |
| `src/__tests__/routes.test.jsx` | 1 | Login, print role, redirect (credential is dead; passes by swallowing the error) |

---

## Related Notes

- [[Known_Issues]]
- [[Refactor]]
- [[Local_Dev_Setup]]
