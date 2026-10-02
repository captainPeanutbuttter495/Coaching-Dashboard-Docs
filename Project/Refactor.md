# Refactor

The pre-handoff refactor: what changed in each phase, and the principle behind each change.

---

## Purpose

**Purpose of this note:** to let anyone — the owner, an advisor, the incoming capstone team — understand
*why* the codebase is shaped the way it is now, and what was wrong with the shape it had before.

This is the explanatory companion to `docs/refactor-phases.md`, which is the operational tracker
(what is done, what is next, what the acceptance criteria were). **This note answers "why".** That
one answers "where are we".

> Live status, deviations, and per-phase verification evidence: `docs/refactor-phases.md`.
> Branch: `refactor`. One phase per work session.

---

## The situation

The project is being handed to a capstone team. It works — it deploys, it authenticates, it reads
real Firestore data — but it grew across roughly a dozen feature branches by different people, and
nobody had ever gone back and tidied it. The goal of this refactor is **not** new features. It is to
make the codebase legible to someone who has never seen it, so their first week is spent building
rather than archaeology.

Every phase carries the same acceptance criterion: **the running app must look and behave exactly
as it did before.** This is a formatting and organization exercise. If a user could tell the
difference, the phase failed.

---

## The seven problems, and the principle that fixes each

### 1. Files were grouped by *what they are*, not by *what they belong to*

**Before:**

```
src/
  Components/           16 loose files + 4 subfolders
  Components/Dashboard/ 16 more
  Pages/                11 route components
  Utils/                4 helpers
  Public/               images
```

Every file in the project was sorted into one of four buckets by its technical kind. That sounds
tidy, and it is the layout most tutorials show, but it fails at the question you actually ask most
often: **"I need to change the coach dashboard — which files is that?"**

Under the old tree the answer was: some of `Pages/`, most but not all of `Components/Dashboard/`,
one file in `Components/`, one hook in `Components/Dashboard/hooks/`, and a helper in `Utils/`. There
was no way to know without reading imports. Worse, `Components/` also held files that *nothing*
imported — abandoned rewrites sitting next to live code, indistinguishable from it.

**After:**

```
src/
  pages/              thin route components, one per URL
  features/           <feature-name>/components, /context, /data, /api
  components/ui/      reusable primitives (DashboardCard, PopupCalendar)
  components/global/  app chrome and route guards (Navbar, Footer, ProtectedRoute)
  lib/                pure helpers with no React in them
  assets/images/
```

**The principle:** *group by feature, not by kind.* A directory name should tell you who owns the
code inside it. `features/coach-dashboard/` is the complete answer to "what is the coach dashboard" —
its twelve cards, its shared timing context, and its data all live under one path. You can delete a
feature by deleting a folder. You can hand a feature to a teammate by naming a folder.

The two exceptions earn their place. `components/ui/` is for genuinely reusable pieces used by more
than one feature — putting them inside any one feature would be a lie about ownership. `lib/` is for
code with no UI at all. The dependency rule that falls out of this: **features may import from `ui`,
`global` and `lib`; those three must never import from a feature.** That single rule prevents the
circular-dependency tangle that the old flat `Components/` folder had no defence against.

### 2. Dead code was indistinguishable from live code

Roughly 10 files — `HeartRateChartNew.jsx`, `SleepTimelineNew.jsx`, `VO2MaxChartNew.jsx`,
`CoachDashboard2.jsx`, and others — were abandoned rewrites that nothing imported. Several live
files carried hundreds of lines of commented-out previous versions: `SleepTimeline.jsx` was 1,170
lines of which 231 ran.

For a newcomer this is worse than clutter. It is actively misleading: you read a `*New.jsx` file,
reasonably conclude it is the current version, and change the wrong thing.

**The principle:** *delete before you organize.* There is no point deciding where a file belongs if
the file should not exist. Git history is the archive — keeping a dead copy in the working tree buys
nothing and costs every future reader. This is why deletion was Phase 1 and restructuring was
Phase 3.

### 3. The build fought the framework's conventions

`vite.config.js` set `root: "src"`, which moved the project's centre of gravity into `src/` and
dragged four other things out of their standard places: the entry HTML, the env file, the build
output, and the public assets directory.

That single line caused three real, user-visible bugs:

- **Deep links 404'd on Netlify** — the SPA fallback was in `public/_redirects`, which never reached
  the build output.
- **A duplicate image shipped** on every deploy, because `src/Public` matched Vite's expected
  `src/public` on Windows' case-insensitive filesystem but not on Netlify's Linux.
- **Firebase initialized by accident.** No file explicitly imported the config; it was pulled in as a
  side effect of `App.jsx → Invitation.jsx → firebaseConfig.js`. Seventeen call sites depended on
  that chain holding. Reordering an import could have broken authentication.

**The principle:** *use the framework's defaults unless you have a reason not to.* Standard layouts
are not merely aesthetic — the tooling, its documentation, and every Stack Overflow answer assume
them. A newcomer who knows Vite already knows where things are. The Firebase fix follows the same
logic: `main.jsx` now imports the config explicitly before rendering, so initialization is a
deliberate, visible step rather than an emergent property of the import graph.

### 4. Import paths were relative, unreadable, and fragile

```js
import DashboardCard from "../../Components/Dashboard/DashboardCard";   // before
import DashboardCard from "@/components/ui/DashboardCard";              // after
```

Relative paths encode *where the importing file happens to sit*, which is information the reader
does not care about and which changes every time a file moves. The `@/` alias encodes where the
target lives, which is what you actually want to know. It also means moving a file no longer
requires editing every file that imports it.

### 5. Mock data was welded into the components that displayed it

The coach dashboard is the most visually finished part of the app and the least real — every card
held its own fake athlete data as literals in the component file, and the page passed no props to
any card. There was no seam through which real data could ever be injected.

That made "wire the dashboard to real data" — the most likely capstone project — read as *rewrite
twelve components*. After Phase 4 it reads as *replace thirteen data modules*.

**The principle:** *separate what a component looks like from what it shows.* Only the mock athlete
content moved. Colour maps, formation geometry, thresholds and axis ranges stayed in the components,
because a real API will never replace those. The folder split is itself the documentation: what is
in `data/` is the incoming team's job; what is not, is not.

### 6. Styling had two systems, and the one people reached for was booby-trapped

Layout lived in a 3,400-line hand-written `style.css`; Tailwind was installed for spot fixes but
configured with a `tw-` prefix, so `className="flex"` — the thing every tutorial, every copied
snippet and every AI assistant produces — silently did nothing. The prefix had been added to keep
utilities clear of daisyUI and Bootstrap, and both of those had already been removed in Phase 1. It
was a lock on a door that no longer existed, and the readiness report listed it as the single most
common mistake in the project.

The stylesheet had its own problems: 600 lines belonged to a card deleted in Phase 1, several rules
were defined two or three times with the later copy silently winning, and one of those later copies
was the only reason the dashboard did not collapse to a single column on every screen.

**The principle:** *one styling system, and the cheap check must be the true one.* The prefix went
first, because it made the tool everyone would reach for lie to them. The dead and duplicated rules
went next. Then two features were converted to utilities outright, with their CSS deleted in the same
edit — and the conversion was proven by comparing every element's computed styles before and after,
because a sorted diff of the CSS text cannot see cascade order, and cascade order was exactly what
the duplicates depended on. As of Phase 8C `style.css` holds nothing but the three `@tailwind`
directives — the hand-written stylesheet is gone, not merely shrinking.

### 7. The tools meant to catch the other six were never wired up

`package.json` had a `lint` script and five ESLint plugins installed, and **no config file** — so
`npm run lint` had never once run. `.prettierrc` was `{}`. There was no `.editorconfig`. The README
told you to open three terminals and start three processes, and there was no script that did it, so
the most common first-day failure — a UI whose every `/api` call 502s — had no guard except a
sentence in a document.

The consequence is subtler than "no linting". The layering rule from problem 1 — *`ui`, `global`
and `lib` never import a feature* — existed only as prose in this note. The 44 `import React` lines
that the automatic JSX runtime had made dead in every component sat there because nothing said so.
And a form on the admin page had been removed while its handler, its state and its four MUI imports
stayed behind — the same class of dead code problem 2 describes, but *created after* problem 2 was
fixed, because nothing stopped it.

**The principle:** *a convention that lives only in a document decays; one the linter enforces
cannot.* Phase 6 wrote the layering rule as six `import/no-restricted-paths` zones and proved each
one fires. It turned on `import/no-unresolved` with `caseSensitive: true`, which catches — on a
Windows checkout, where it was previously invisible — the exact `src/Pages` casing bug that broke the
Netlify build twice. And it ran Prettier across the code once, so that the first capstone pull
request does not carry a whitespace diff for someone else's file.

The number to remember is **0 errors, 39 warnings**, and why the 39 are warnings: every one is an
accessibility or hook-dependency finding whose fix changes behaviour, and this refactor does not
change behaviour. Leaving them visible, with the config saying in a comment which ticket owns them,
is more honest than either suppressing them or fixing them in a phase whose acceptance is "the app
is the same".

---

## Phase log

Each phase is one work session, ending with a handoff report the owner reviews before committing.

| Phase | What | Status |
| --- | --- | --- |
| 0 | Branch and baseline | Done |
| 1 | Delete dead code and dependencies | Done — `b539ba3` |
| 2 | Normalize the Vite layout, fix the deploy bugs | Done — `45f4258` |
| 3 | Restructure into `features/` | Done — `4fa4ca3` |
| 4 | Extract the coach dashboard's static data | Done — `28a8a25` |
| 5 | Styling pass — drop `tw-`, delete dead CSS, convert Settings + Athlete Access | Done — `d9c5d92` |
| 6 | Tooling — ESLint, Prettier, `.editorconfig`, dev scripts | Done — `99a9eb6` |
| 7 | Documentation refresh | Next |
| 8A | Tailwind conversion — team insights | Done — `d9c5d92` |
| 8B | Tailwind conversion — coach dashboard shell + header | Done — `1374879` |
| 8C | Tailwind conversion — team builder + sport art | Done — `5ae5ae6` |
| 8D | Inline `style={{}}` objects — `dashboardCardStyle` and the 12 cards | Done — `5ae5ae6` |
| — | Mop-up: inline styles on the remaining routes | Backlog |
| — | Make `npm run lint` warning-free; toolchain upgrade | Backlog |

### Phase 1 — Delete dead code and dependencies

Removed 10 orphan files and 3.2 MB of unreferenced images. Stripped commented-out prior versions
from four live files. Dependencies fell from **38 → 29** runtime and **25 → 15** dev; installed
packages from 989 to 547.

*Why it matters:* smaller install, faster builds, and — more importantly — every remaining file is
now load-bearing. If you see it, it runs.

### Phase 2 — Normalize the Vite layout

Dropped `root: "src"` and let the four consequences follow: `index.html` and `.env.local` at the repo
root, build output to `dist/`, SPA fallback moved into `netlify.toml`. Added the `@/` alias. Split
`main.jsx` (mount + explicit Firebase init) from `App.jsx` (routes only). Committed `.env.example`
and a service-account template so a fresh clone can be configured without guesswork.

*Why it matters:* fixed three deployment bugs and made the project's shape match what any Vite
developer expects.

### Phase 3 — Restructure into features

47 files moved; 32 files had their imports rewritten. **No file body changed except import
specifiers** — this was deliberate, so that reviewing the commit means checking a file list rather
than re-reading the whole codebase.

*Why it matters:* this is the phase that produced the layout described above, and the one a newcomer
will feel most directly.

*Resolved before Phase 5:* the case-only rename that Windows' filesystem hid from git was fixed in the
index and committed as `d7443a9`, so a Linux checkout now gets `src/pages/…` and the Netlify build resolves.

### Phase 4 — Extract the static dashboard data

13 new data modules under `features/coach-dashboard/data/` and
`features/athlete-access/data/`; 14 files rewired to import from them. Cards take the data as a prop
defaulting to its module, so nothing needed to change at the call site.

Correctness was *proven*, not eyeballed: every one of the twelve cards was rendered in both its
pre-change and post-change form and asserted to produce byte-identical markup.

Cataloguing the data also surfaced eight pre-existing defects — an empty roster filter, several
hardcoded numbers that masquerade as computed values, inert buttons on `/athlete-access`. None were
fixed, because each would change what renders and break the phase's acceptance. They are listed in
`docs/refactor-phases.md` and belong in `tickets/BACKLOG.md`.

### Phase 5 — Styling pass 1

Dropped the `tw-` prefix (218 tokens, nine components), deleted 37 dead classes — mostly the
stylesheet of a card Phase 1 had already removed — collapsed three sets of duplicated rules, and
converted the Settings cluster and Athlete Access to Tailwind, deleting their rules as they went.
`style.css`: 3,398 → 2,131 lines. The project's nine real breakpoints are now named max-width screens
in `tailwind.config.js`.

The plan changed at the table: the original sketch split `style.css` into a per-feature tree and
added `:root` design tokens. Both were dropped because the destination is Tailwind, not tidier CSS —
and keeping the file in place means deleting rules can never reorder the cascade.

Correctness was proven in a browser, not by diffing text: full-page computed-style dumps of every
route at three widths, before and after each step, all identical (33/33, 33/33, 22/22). That method
caught two real regressions before they landed — deleting the *earlier* of two identical rules when
a responsive rule sat between them, and a renamed override that matched any element carrying
`flex justify-between`. Both are written up in `docs/refactor-phases.md` because Phase 8 will meet
them again.

*Why it matters:* `className="flex"` now does what every Tailwind tutorial says it does, the
capstone team has a worked example of a feature converted without changing a pixel, and the biggest
styling footgun in the readiness report is gone.

### Phase 8A — Team insights converted (pulled forward)

Phase 8 was originally deferred to the capstone team. The owner pulled it forward so the handoff ships
one styling system instead of two. It runs as three steps; 8A is team insights — 63 rules gone,
`style.css` 2,132 → 1,718 lines, verified by 105 computed-style captures with a measured noise floor of
zero.

*Why it matters, beyond the line count:* 8A removed a config that was quietly lying to everyone.
Phase 5 declared the project's nine breakpoints as named `to*` screens, which read as the tidy
solution. They were worse than useless. Declaring `screens` in object form makes Tailwind **disable
the `min-*`/`max-*` variant family entirely** — it warns once at build time and thereafter emits
nothing for `max-[768px]:…`, with no error where you wrote it. And named max-width screens stack in
the wrong order regardless of how you write the config, so a two-breakpoint rule silently kept the
wider value. Both were found by measurement rather than reading: the first conversion came back with
the page unstyled below 768px, and the fix was to delete the config and write `max-[900px]:` inline.

The deeper lesson is the one Phase 5 already recorded and 8A confirmed twice over: **the cheap check
has to be the true one.** `getComputedStyle` normalises `rgba(45,212,191,0.45)` and
`rgb(45 212 191 / 0.45)` to the same string, so the dumps said "identical" while every bordered
element rendered a 1px ring one shade off. Screenshots caught what the dumps could not — and equally,
screenshots on `/coach` are pure noise (30 of 105 differ between two runs of the same code) where the
dumps are exact. Neither instrument is sufficient alone, and knowing which one to believe for a given
route is the actual skill.

### Phase 8B — Coach dashboard shell and header converted

The middle step of Phase 8. `style.css` 1,717 → 1,354 lines; two files converted
(`pages/CoachDashboard.jsx`, `features/coach-dashboard/components/CoachDashboardHeader.jsx`); 105/105
computed-style dumps identical against a noise floor of zero, and the header band pixel-identical at
all five widths. What is left in the stylesheet is now exactly one feature: the team builder.

*Why it matters, beyond the line count:* this phase is where a **known bug had to be deliberately
preserved**, and being able to say why is the point. The card grid was declared after its own
responsive rules, so the two-column layout won at every width and the dashboard never collapsed —
the `@container` query and both `@media` blocks were fossils that had never once applied. The
conversion could have fixed that for free by simply writing the utilities the CSS *looked* like it
intended. It did not, for two reasons. First, "make the dashboard responsive" is a design decision
about what a coach should see on a narrow screen, and nobody has made it; smuggling it into a
mechanical conversion would hide a product change inside a refactor commit. Second, and more
practically, the entire acceptance method rests on the conversion being a provable no-op — the
moment layout changes on purpose, 0 diffs stops being the bar and there is nothing left to check the
rest of the work against. So the fossils were deleted, the result they never produced was kept, and
the bug moved from invisible (buried in cascade order) to obvious (written in the JSX with a comment
explaining it). A refactor's job is to make the next decision easy, not to make it silently.

The secondary lesson is about instruments again. Phase 5 and 8A both established that
`getComputedStyle` can normalise away a real visual difference; 8B found the converse — Tailwind's
`items-start` emits `flex-start` where the stylesheet said `start`, two keywords that render the
same and dump differently. Reading the *existing dumps* before writing the conversion, rather than
translating the CSS source and diffing afterwards, caught that and two others before a single edit.
And the four `:hover` rules, which no resting dump can see and which 8A converted "by inspection",
were this time forced through CDP and compared to the deleted declarations: 7 values, 7 matches.

### Phase 8C — Team builder and sport art converted; `style.css` reaches zero

The last conversion. `style.css` 1,354 → **3 lines** (the `@tailwind` directives), from one file:
every remaining rule belonged to `MiniTeamBuilderCard.jsx` alone. 115/115 computed-style dumps
identical against a re-measured noise floor of zero, all four sport surfaces pixel-identical at all
five widths, and 28 forced `:hover` / `:active` / `:disabled` states matching the declarations they
replaced.

*Why it matters, beyond the line count:* the file the project started with is now empty, so the
class of bug that shaped Phases 5 through 8B — a hand-written rule quietly beating a utility because
`@tailwind utilities` sits at the top of the file — **cannot happen any more**. That is the actual
deliverable. The line count is how you can tell.

The phase also produced the clearest instance yet of the argument this whole refactor rests on:
*the stylesheet is not evidence of what the page looks like.* Three examples, all caught by reading
the baseline dumps rather than the CSS:

- The "New Team" button declares `padding: 8px 13px`, `font-size: 12px`, `font-weight: 950`. It has
  never used any of them. `.mini-team-controls button` is a descendant selector and outranked it, and
  the three `!important` flags the author added rescued only the colours — which is itself the
  fingerprint of someone fighting this exact cascade and not realising how much they had lost.
- The position badge renders at font-weight 800 while its own rule says 950, because it sat under
  three overlapping selectors at once and a different one won each property.
- `.sport-basketball::before` was declared twice, 23 lines apart, with different `z-index` values.
  Converting from the first block would have put the court's lighting overlay *above* its lines.

A faithful conversion had to reproduce all three, so the phase records them as findings rather than
fixing them — same discipline as 8B's card grid, and for the same reason: the moment the conversion
changes what renders on purpose, "0 diffs" stops being a bar and there is nothing left to check the
rest of the work against.

The new lesson is about the tool rather than the CSS, and it is a nasty one. **Tailwind scans source
text, not evaluated JavaScript**, so a class assembled by concatenation — `"[border:2px_solid_" +
WHITE + "]"` — leaves no complete class name anywhere in the file and emits *nothing at all*, with
no warning and no error. That silently deleted every white line on the basketball court. A dump
caught it; nothing else would have. The same property makes the old `` `avatar-${initials}` ``
pattern unusable, which is why the runtime variants now live spelled out in
`teamBuilderClasses.js` — a file that exists solely because Tailwind cannot see through an
interpolation.

The second new trap is that **two arbitrary properties on one element resolve by Tailwind's internal
sort order, not the order you wrote them in**: `[border-top:0]` sorts before
`[border:1px_solid_…]`, so the shorthand won and a deliberately open box side grew its line back.
Both bugs were in the same step, both were invisible to review, and both showed up as exactly the
kind of small, specific diff the capture harness exists to produce.

What is deliberately *not* in this phase: the ~280 inline `style={{}}` objects, led by
`dashboardCardStyle` in `components/ui/DashboardCard.jsx` that all twelve cards spread. 8B already
declined to convert that seam once, to avoid leaving the KPI row styled one way and the cards
beneath it the other. It is now Phase 8D, with its own verification pass, because it is a different
kind of work: inline styles never fight the cascade, so the risk profile and the evidence needed are
both different.

### Phase 8D — The coach dashboard's inline styles converted

**185 in-scope `style={{}}` objects → 23**, across 14 files, verified the same way as 8C: 115/115
computed-style dumps identical at each of four steps, against a re-measured noise floor of zero.

*Why it matters, beyond the count:* this is the phase where the honest answer was **"not all of
them"**, and being able to say why is the whole point. Twenty-three objects remain and every one is
a value the framework cannot express: a colour that comes out of a data module, a bar width computed
from a percentage, a module constant, a CSS custom property driving a keyframes animation. Tailwind
matches **class names in source text**, so the moment a value is interpolated there is no class to
find. A phase that reported "inline styles eliminated" would have had to either inline the data into
the markup or invent a lookup keyed on colour strings — both worse than what is there now. The
useful summary is *everything a class can express is a class*.

The second thing worth keeping is a lesson about **what a refactor is allowed to retype**.
`TopRiskAthletesCard` picks a badge colour with a three-level nested ternary over athlete data. The
conversion lifted the static properties out around it and **moved the expression as source text**,
never re-typed it. Retyping is how a mechanical change smuggles in a behaviour change — and unlike a
styling mistake, no computed-style dump can distinguish a mis-typed ternary from an intended one,
because both produce a page that renders exactly as the new code says it should. The tooling only
protects the half of the work it can see.

Third: **the evidence had to be graded rather than trusted.** 8C could pixel-compare its whole card
because the team builder contains no chart. Most of these cards do, and Recharts rasterizes
differently between two runs of identical code. So each card's bounding box was compared against
itself first, and only the 29 of 40 regions with a zero noise floor were treated as evidence — 27 of
which came back pixel-identical. Two outliers survived that filter and were cleared by *attribution*:
one differs by 49,619px between two identical runs, and the other appears across a step where its own
file did not change while being absent across both steps where it did. Neither could be waved away,
and neither turned out to be real.

A by-product worth recording: at 800 and 900px, four card bounding boxes fall partly outside the
viewport, so they cannot be pixel-compared at all. The card-grid overflow carried since Phase 5 has
stopped being a cosmetic complaint and started destroying evidence. See [[Coach_Dashboard]].

What this leaves: about 35 inline styles on routes outside the coach dashboard - `SleepTimeline`,
`PopupCalendar`, `Admin`, `Login`, `Register`, the settings pages. They are a mop-up ticket rather
than a phase, and deliberately so: `/admin` and `/athlete` have no capture coverage because no login
exists for either role, so converting them would mean shipping part of the work verified only by
inspection. That is a decision about what evidence is worth, not about effort.

### Phase 6 — Tooling

`npm run lint` ran for the first time in the project's life: 0 errors, 39 warnings. Four config
files added at the repo root, the `dev:all` / `lint:fix` / `format:check` / `test:run` scripts,
46 files with unused imports and bindings removed, 55 files formatted. No toolchain version changed —
the ESLint 9 / Prettier 3 / Vite 5 migration is a separate ticket, on purpose.

*Why it matters, beyond the config:* this is the phase that turns problem 7 from a description into
a mechanism. The layering rule was tested **negatively** — six temporary imports that break it, each
producing the intended error with its own message, then reverted — because a rule nobody has seen
fire is a rule nobody can trust. The casing check was tested the same way and produced *"Casing of
@/Pages/Login does not match the underlying filesystem"* on the machine where that spelling used to
work fine.

Two decisions are worth being able to defend. First, **the warnings were left as warnings.** Fixing
`click-events-have-key-events` means adding key handlers; fixing `no-autofocus` means removing a
focus the login page relies on; fixing `exhaustive-deps` means changing when an effect re-runs. Each
is a product change, and this refactor's acceptance has been "the app is the same" since Phase 0.
The config says so in a comment next to each downgraded rule, with the ticket that owns it. Second,
**Prettier was run now rather than merely configured.** The alternative leaves `npm run format` as
a landmine for whoever touches a file first. Running it was made safe by `--debug-check`, which
formats in memory and refuses if the syntax tree changes — and then confirmed by building before and
after and finding the shipped JavaScript identical except at three places where Prettier had written
a trailing JSX space as `{" "}`, which is the same DOM text.

The evidence changed shape this phase, and that is worth noting for the team. Phases 5 through 8D
proved "nothing changed" with computed-style dumps in a browser. Here the check was the **compiled
bundle**: removing unused imports and reformatting cannot alter a computed style, and a byte
comparison of what Vite emits is both cheaper and stronger for that kind of change — it proves the
shipped code is the same, which is more than a style dump can say. Knowing which instrument fits
which change is the skill; the dumps are not the answer to everything.

What the linter found on day one, recorded rather than fixed: the admin page's dead "promote to
coach" form, which means **nothing in the UI calls `/api/updateRole` any more**; a
`PendingInvitations` component that ignores the data its page fetches for it; and a `HeartGraph`
`userId` prop that was accepted and never read. None of these needed a linter to find — but none had
been found without one.

---

## How to update this note

**At the end of each phase**, before the owner commits:

1. Flip the phase's row in the table above, and mark the next phase `Next`.
2. Add a short subsection under *Phase log* — a few sentences on what moved and, crucially, a
   *"why it matters"* line. Resist writing a changelog; `docs/refactor-phases.md` already is one.
   This note earns its place only by explaining reasoning.
3. If the phase changed one of the seven problems described above — or revealed a sixth — update that
   section. The argument is the point of the note; the log is supporting evidence.
4. Update the matching note in `Ticket/` (one per phase) and its row in [[Ticket/README|the ticket index]].
5. If the phase invalidated another vault note, fix it too. Phase 5 rewrote [[Styling_And_Charts]]'s
   prefix and `style.css` sections the same day the prefix was removed.

---

## Caveat: This Vault Is Not in Git

`coaching-dashboard-documentation/`, `docs/` and `tickets/` are all excluded by `.gitignore`. This
note exists only on the owner's machine and will not reach the capstone team through a clone. If it
is meant to be part of the handoff, that needs deciding — see the open question in
`docs/refactor-phases.md`.

## Related Notes

- [[Project_Structure]]
- [[04-Component-Frontend]]
- [[Frontend_Data_Flow]]
- [[Coach_Dashboard]]
- [[Styling_And_Charts]]
- [[Known_Issues]]
- [[Ticket/README|Ticket index]]: one note per phase, with resume bullets
