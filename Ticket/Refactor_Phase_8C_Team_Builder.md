---
title: "Refactor Phase 8C: Team Builder and Sport Art"
date_started: 2026-09-14
date_finished: 2026-09-14
repo: coaching-dashboard
area: refactor
branch: refactor
pr:
commit: 5ae5ae6
status: committed
standups: []
skills: [tailwind, css, visual-regression-testing]
tags: [ticket]
---

# Refactor Phase 8C: Team Builder and Sport Art

Converted the last 1,354 lines of hand-written CSS (the mini team builder and its sport-surface art) to utilities. `src/style.css` is now three `@tailwind` lines.

---

## Ticket

Refactor Phase 8, third step. Scope re-cut at planning: the inline `style={{}}` objects split off into [[Refactor_Phase_8D_Inline_Styles]]. See [[Refactor]].

## What I Did

- Converted `MiniTeamBuilderCard.jsx` (~100 `className` sites). **`style.css` 1,354 → 3 lines.**
- New `teamBuilderClasses.js` spelling out every runtime variant (avatar colours, sport surfaces, position badges, status dots) in full.
- Deleted four rule sets that were provably unreachable; kept the backgrounds for three locked sports so unlocking one later does not ship a surface with no art.

## Technical Details

- **A class built by concatenation emits nothing.** `"[border:2px_solid_" + WHITE + "]"` leaves no complete class in the source text, so Tailwind generated nothing and every white line on the basketball court vanished. Fixed by writing each class in full, plus a regex guard.
- **Two arbitrary properties resolve by Tailwind's sort, not the order written.** `[border-top:0]` lost to `[border:…]`, reopening intentionally open box sides. Fixed with four longhands.
- The baseline dumps revealed rules that never applied (a button that never used its own padding, a badge at weight 800 not 950, a `z-index` declared twice), and the conversion preserved what actually rendered.
- Confirmed four utilities are not the CSS they look like (`outline-none`, `grid-cols-1`, `rounded-full`, `border-0`) in a compiler probe before touching JSX.

## Testing

- Computed-style dumps: **115/115 identical** (23 states × 5 widths, `::before`/`::after` as their own rows). Noise floor 0.
- Region pixel diffs: 37 of 40 regions 0 differing pixels; the three exceptions reproduced on identical code.
- **28 forced pseudo-states** match the deleted declarations.
- Build 2,554 modules. `wc -l src/style.css` is 3.

## Impact

The project has one styling system. There is no stylesheet left to fight.

## Resume Bullets

- **Completed a full migration from a 3,398-line hand-written stylesheet to Tailwind CSS** across four phases with zero measured rendering differences (115/115 computed-style captures identical in the final phase).
- **Found two silent Tailwind failure modes** (interpolated class names and arbitrary-property sort order) through automated comparison, and documented them as team conventions.

## Follow-ups / Open Questions

- Team builder defects left untouched: empty roster tabs, two divergent 4-3-3 layouts. See [[Known_Issues]].
- The capture harness lives in session scratchpads, not the repo. Whether the capstone team gets it is the owner's call.

---

## Related Notes

- [[Refactor]]
- [[Styling_And_Charts]]
- [[Coach_Dashboard]]
