# Styling and Charts

How the SPA is styled (Tailwind utilities, nothing else), the conversion traps the refactor measured, and which chart library goes where.

---

## Overview

Styling is **Tailwind utilities in `className`, unprefixed**. `src/style.css` is three `@tailwind` lines; there is no hand-written stylesheet to add a rule to. The conventions are stated tersely in `AGENTS.md` → "Styling". This note carries the reasoning and the traps behind them, because the same traps apply to any future styling work.

---

## Tailwind Is Unprefixed

Refactor Phase 5 (2026-09-07) removed `prefix: "tw-"` from `tailwind.config.js`. Every utility is
now the stock Tailwind name:

```jsx
<div className="flex items-center gap-4">      // correct
<div className="tw-flex tw-items-center">       // does nothing — the prefix is gone
```

The prefix existed to keep utilities from colliding with daisyUI and Bootstrap component classes.
Both libraries were removed in Phase 1, so the prefix had become pure cost: `className="flex"`
silently did nothing, and every snippet copied from the Tailwind docs was wrong by default.

**Breakpoints — write `max-[900px]:`, not a named screen.** The hand-written CSS uses max-width
queries at 700/768/850/900/950/1000/1150/1200/1700px, none of which are Tailwind's defaults. Phase 5
declared them as named `to*` screens; **Phase 8 removed that config entirely**, because it turned out
to be actively harmful in two ways:

- A `screens` config **in object form** (`{ max: "900px" }`) makes Tailwind disable the whole
  `min-*`/`max-*` variant family. It prints `warn - The min-* and max-* variants are not supported
  with a screens configuration containing objects` and then silently emits nothing for
  `max-[900px]:grid-cols-1`. You get no error at the call site, just unstyled output.
- Named max-width screens **stack in the wrong order**. Tailwind sorts screen variants by breakpoint
  value ascending regardless of how the config is written — reordering the object produces
  byte-identical CSS — so `to1200:grid-cols-2 to900:grid-cols-1` left two columns at 800px.

`max-[1200px]:grid-cols-2 max-[900px]:grid-cols-1` orders correctly and puts the pixel value at the
call site. Use it rather than `md:`/`lg:` when reproducing an existing responsive rule.

**Converting an `rgba()` colour? Use an arbitrary property.** `border-teal-400/[0.45]` emits modern
`rgb(45 212 191 / 0.45)`. `getComputedStyle` reports that identically to the stylesheet's legacy
`rgba(45,212,191,0.45)`, but Chrome rasterizes the two one 8-bit step apart on composited edges — a
faint 1px ring around every bordered element. Write `[border-color:rgba(45,212,191,0.45)]` instead.

---

## `style.css` Is Gone

`src/style.css` is imported once, from `src/main.jsx`, and as of **Phase 8C it contains nothing but
the three `@tailwind` directives**. The hand-written stylesheet this project started with is fully
converted: there is no longer a rule anywhere to add to, override, or accidentally inherit from.

The arc, for anyone reading the history: 3,398 lines → Phase 5 → 2,131 → Phase 8A (team insights) →
1,718 → Phase 8B (coach dashboard shell and header) → 1,354 → Phase 8C (team builder and sport art)
→ **3**.

Everything below is kept because the *lessons* outlive the file, and the same traps apply to any
future conversion. Phase 8D has since converted the coach dashboard's inline `style={{}}` objects
(185 → 23 irreducible, data-driven ones); about 35 stragglers remain on other routes.

- **A surviving hand-written rule beat a utility.** `@tailwind utilities` is emitted at the top of
  the file, so any later rule of equal specificity won. Every phase converted a component and deleted
  its rules in the same edit; a half-converted component silently kept the old styling. This is no
  longer possible, but it is why the phases were shaped the way they were.
- **Order matters even for identical rules, and a later rule can strand a whole `@media` block.**
  The coach-dashboard card grid used to be declared *after* its own `@container (min-width: 1720px)`
  and `@media (max-width: 1700px)` / `(max-width: 850px)` rules, so the unconditional two-column
  declaration won at every width and those three rules had never once applied. Phase 5 kept the later
  copy for exactly that reason. Phase 8B converted the grid and deleted the fossils, **preserving the
  two-column result rather than fixing it** — the dashboard still does not collapse below 1700px, and
  it now overflows visibly below ~900px. That is a live design decision, not an oversight; see
  [[Refactor]]. The general lesson survives the specific case: before deleting a "duplicate" rule,
  check what sits between the copies.

Verification that a styling change is a no-op is done in a real browser, not by diffing CSS text:
full-page `getComputedStyle` dumps per state at 1920/1440/900/800/768px, before and after, against a
re-measured noise floor. The Phase 5, 8A, 8B and 8C records in `docs/refactor-phases.md` describe the
method and what each phase learned from it.

---

## Tailwind Traps That Cost Real Time

Four of these were found by a dump or a pixel diff catching a conversion that *looked* right. They
are worth reading before writing utilities against an existing design.

- **A class built by interpolation emits nothing.** Tailwind scans source text, not evaluated
  JavaScript. `` `avatar-${initials}` `` and `"[border:2px_solid_" + colour + "]"` both leave no
  complete class name anywhere in the file, so Tailwind generates nothing — with no warning. In 8C
  the second form silently deleted every white line on the basketball court. Runtime variants belong
  in a lookup object with each variant written out in full:
  `src/features/coach-dashboard/components/teamBuilderClasses.js`.
- **Two arbitrary properties on one element resolve by Tailwind's sort, not yours.**
  `[border-top:0]` sorts before `[border:1px_solid_…]`, so the shorthand won and an intentionally
  open box side grew its line back. Write four longhands rather than shorthand-plus-override.
- **Several keyword utilities are not the CSS keyword.** `outline-none` is
  `outline: 2px solid transparent`; `grid-cols-1` is `repeat(1, minmax(0, 1fr))`, whose zero minimum
  differs from `1fr` once content overflows; `rounded-full` is 9999px, not 999px; `border-0` zeroes
  the width but keeps preflight's `solid` and grey, where `border: 0` resets style and colour too;
  `items-start` emits `flex-start`, not `start`.
- **`calc()` needs its spaces escaped** — `top-[calc(15%_-_60px)]`. `calc(15%-60px)` is invalid CSS
  and emits nothing.
- **An interpolated value can never be a class.** Tailwind matches class names in source text, so
  `` `w-[${pct}%]` `` and `"[background:" + colour + "]"` both emit nothing. This is the same
  property that makes runtime variants need a lookup object - and it is also why converting inline
  styles stops where the data starts. A value computed at runtime belongs in a `style` object; that
  is not technical debt, it is the correct tool.
- **Verify that a class EMITS before trusting it.** Compile the candidate strings through the
  Tailwind CLI and check each one appears in the output. Phase 8C lost every white line on the
  basketball court to a class that silently generated nothing; Phase 8D ran this as a pre-flight
  over all 195 of its generated classes and caught nothing, which is what the check is for.
- **Keep `rgba()` notation.** Slash-opacity emits the modern `rgb(r g b / a)` form, which
  `getComputedStyle` reports identically but Chrome rasterizes one 8-bit step apart on composited
  edges. Irrelevant to the eye; it matters because it is what keeps screenshots usable as evidence.

---

## Reading a Computed-Style Dump

- **Blockification.** An `inline-grid` element that is a flex or grid item computes as `grid`; an
  `inline-flex` one computes as `flex`. The declaration and the computed value legitimately differ.
- **Specificity beats intent, and the dump is the only honest witness.** The team builder's position
  badge sat under three overlapping rules at once and rendered at font-weight 800 while its own rule
  said 950. Its "New Team" button has never once used the padding, font-size or font-weight it
  declares, because a descendant selector outranked it. Read the value out of the baseline; do not
  reason it out from the stylesheet.
- **A screenshot is not always a signal.** Whole-page `/coach` captures differ between two runs of
  identical code (Recharts and Chart.js rasterize differently run to run), so pixel equality is only
  meaningful inside a region that contains no chart, and only against a re-measured noise floor.

**Trust the dumps, not the screenshots.** Phase 8A measured this: 30 of 105 screenshots differ between
two runs of *identical code*, because Recharts and Chart.js rasterize differently run to run (pinning
scroll to 0 does not fix it). The computed-style dumps over the same 105 captures have a noise floor of
**zero**. Screenshots are still worth keeping as supporting evidence for chart-free routes — every
team-insights capture is byte-identical across 8A — but on `/coach` they cannot prove anything.

---

## Charting: Two Libraries, Split by Era

Two are used, and which one depends on when the component was written.

| Library | Used by | Era |
|---------|---------|-----|
| **Recharts** | `TeamInjuryRisk`, `TeamLoadZonesCard`, `TeamSleepDurationCard`, `TeamVo2TrendCard` (coach dashboard), `SleepTimeline` (athlete health) | newer |
| **Chart.js** via `react-chartjs-2` | `ActivityRing`, `HeartGraph`, `Vo2MaxChart` (athlete health), `MultiHeartGraph` (team insights) | older |
| hand-built | `SleepStageGraph` (team insights) | — |

Chart.js comes with three plugins: `chartjs-adapter-date-fns` (time axes), `chartjs-plugin-annotation`, and `chartjs-plugin-zoom`.

**Check what a component already uses before adding a chart.** Mixing the two inside one card means two rendering models and two theming systems in the same box. Both also rasterize slightly differently run to run, which is why whole-page screenshots of `/coach` are not usable evidence.

## UI Component Libraries

Phase 1 removed Bootstrap, daisyUI, Material Tailwind and `react-pro-sidebar`. What remains:

| Library | Used by |
|---------|---------|
| MUI (`@mui/material`, `@mui/x-data-grid`) + Emotion | `Admin` (data grid), `Invitation`, `Login`, `Register` |
| FontAwesome, `react-icons` | icons in a handful of components |

If you are adding UI, match the file you are editing rather than introduce another library.

## Runtime `<style>` Injection

Two components inject a global `<style>` block while mounted: `PopupCalendar` (its `.day`, `.sidebar`, `.nav-btn` rules) and `SyncedSegmentedToggle` (two `@keyframes`). They are the only CSS in the app outside Tailwind, and they are invisible from the outside. `.sidebar` and `.day` are generic enough to collide with anything added later.

## Images

The one bundled image is `src/assets/images/ARCS.jpg`, imported as an ES module (`@/assets/images/ARCS.jpg`) so Vite fingerprints it. There is no `public/` directory. Phase 2 moved the old `src/Public/` here, which also fixed a case-sensitivity bug that duplicated images in the build.

---

## Key Files

| File | Purpose |
|------|---------|
| `src/style.css` | The three `@tailwind` directives |
| `tailwind.config.js` | Content globs only: no `screens`, no prefix |
| `src/features/coach-dashboard/components/teamBuilderClasses.js` | Runtime variants spelled out in full |
| `src/features/team-insights/components/graphClasses.js` | Shared graph header classes |
| `src/components/ui/DashboardCard.jsx` | `DASHBOARD_CARD_BASE`, the shared card chrome |

---

## Related Notes

- [[Coach_Dashboard]]
- [[Team_Insights]]
- [[Testing]]
- [[Refactor]]
- [[Refactor_Phase_8C_Team_Builder]]
