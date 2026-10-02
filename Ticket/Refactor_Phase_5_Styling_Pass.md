---
title: "Refactor Phase 5: Styling Pass 1"
date_started: 2026-09-07
date_finished: 2026-09-07
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: d9c5d92
status: committed
standups: []
skills: [tailwind, css, browser-automation, visual-regression-testing]
tags: [ticket]
---

# Refactor Phase 5: Styling Pass 1

Dropped Tailwind's `tw-` prefix, deleted dead and duplicated CSS, and converted Settings and Athlete Access to utilities, with every step proven a no-op by computed-style dumps in a real browser.

---

## Ticket

Refactor Phase 5. **Scope re-cut at planning by owner decision:** the original plan split `style.css` into per-feature files with `:root` tokens; that was dropped because the destination is Tailwind, and a token layer is infrastructure for CSS that is going to be deleted. See [[Refactor]] problem 6.

## What I Did

- **A. Dropped the `tw-` prefix.** 218 tokens rewritten across nine components. The prefix only existed to avoid daisyUI/Bootstrap collisions, and both left in Phase 1.
- **B. Deleted dead CSS.** 37 dead classes, 68 rules, 5 selector lists pruned, 3 emptied `@media` blocks: 3,398 → 2,744 lines. Deadness recomputed with strict criteria (zero whole-word hits, not template-constructible, not data-driven, not a descendant of a live rule).
- **C. Converted Settings and Athlete Access.** Eight components; 73 rules removed: 2,744 → 2,131 lines.

## Technical Details

Verification ran Brave through the devtools MCP with the 6.5s auto-cycle frozen, at 1440/900/768px, dumping 27 computed properties on every element keyed by structural path. Noise floor: 0 differing elements between two same-state captures.

**Two regressions caught before landing:** collapsing duplicated card-grid rules by keeping the *earlier* copy changed 144 elements on `/coach` (a `@media` rule sits between the copies); and the prefix removal let a global `.flex.justify-between` override grab the converted Athlete Access rows at 768px, the exact collision the prefix used to hide.

## Testing

- Computed-style dumps: **Step A 33/33 identical, Step B 33/33, Step C 22/22.**
- `npm run build` same 2,552 modules; CSS 52.27 → 44.49 KiB. `npx vitest run` 5/5.
- Not verified: hover states, `/admin` and `/athlete` (no logins for those roles).

## Impact

`style.css` 3,398 → 2,131 lines, and `className="flex"` works like the Tailwind docs say it does.

## Resume Bullets

- **Built a visual-regression method for a CSS refactor:** full-page computed-style dumps across three viewports in an automated browser, which caught two regressions that visual review had missed.
- **Removed 1,267 lines of legacy CSS** (37% of the stylesheet) with zero measured rendering differences.

## Follow-ups / Open Questions

- The coach dashboard grid never collapses: its responsive rules were fossils. Kept as-is; see [[Known_Issues]] #17.
- The named `to*` screens added here turned out to be harmful and were removed in [[Refactor_Phase_8A_Team_Insights]].

---

## Related Notes

- [[Refactor]]
- [[Styling_And_Charts]]
- [[Testing]]
