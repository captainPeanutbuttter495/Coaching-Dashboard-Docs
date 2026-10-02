---
title: "Refactor Phase 6: Tooling"
date_started: 2026-09-14
date_finished: 2026-09-14
repo: coaching-dashboard
area: tooling
branch: refactor
pr:
commit: 99a9eb6
status: committed
standups: []
skills: [eslint, prettier, developer-experience, static-analysis]
tags: [ticket]
---

# Refactor Phase 6: Tooling

Made `npm run lint` run for the first time (0 errors), turned the folder layering rule into a lint error, formatted the codebase with Prettier, and added a one-command dev script that fails loudly.

---

## Ticket

Refactor Phase 6. See [[Refactor]] problem 7 ("the tools meant to catch the other six were never wired up").

## What I Did

- Added `.eslintrc.cjs`, `.prettierrc` (explicit options), `.prettierignore`, `.editorconfig`; two devDependencies (`eslint-import-resolver-alias`, `concurrently`).
- Encoded the Phase 3 layering rule as six `import/no-restricted-paths` zones, and set `import/no-unresolved` to `caseSensitive: true`.
- Fixed 75 `no-unused-vars` errors **mechanically, by category**: unused imports deleted (including 44 `import React`), unused trailing params deleted, dead locals deleted, unread siblings renamed `_x`.
- Left 39 behaviour-changing findings (37 `jsx-a11y`, 2 `exhaustive-deps`) as visible **warnings**, with the rules downgraded and a comment naming the backlog ticket.
- Ran Prettier across 55 files as the last step, and added `dev:all`, `lint:fix`, `format:check`, `test:run`.

## Technical Details

- Legacy `.eslintrc` format, not flat config, because `eslint@8.24` and `eslint-plugin-react-hooks@4` are pinned. Upgrading is its own ticket; no version changed here.
- `dev:all` uses `concurrently -k`, so an API that cannot bind ends the session instead of leaving a UI that 502s.

## Testing

- `npm run lint`: **0 errors, 39 warnings.**
- The rules were **tested negatively**: six temporary probes (lib→feature, ui→feature, feature→page, wrong-case `@/Pages/Login`, a missing module, and a legal control) each produced exactly the expected result.
- Lint-fix gate: build green at 2,554 modules; CSS byte-identical; bundle diffed with identifiers normalized, **46 differing lines, each attributable to a deletion**.
- Prettier gate: `--debug-check` on all 70 files before writing; the bundle differs in exactly 9 minified lines at 3 sites, all JSX whitespace expressed as `{" "}`.
- `npm run dev:all` live, plus a second instance to prove `EADDRINUSE` kills the session.

## Impact

The layering rule and the case-sensitivity bug that broke two earlier phases are now caught by the linter, on Windows, before a push.

## Resume Bullets

- **Introduced ESLint and Prettier to an unlinted React/Express codebase**, reaching 0 errors by fixing 75 findings mechanically and proving the shipped bundle unchanged by diffing it.
- **Encoded an architecture rule as lint errors** (import direction between layers, case-sensitive paths) and verified each rule with negative tests.

## Follow-ups / Open Questions

- Make lint warning-free; upgrade the end-of-life toolchain. See [[Roadmap]].
- Found: nothing in the UI calls `/api/updateRole` any more; `PendingInvitations` ignores its prop.

---

## Related Notes

- [[Refactor]]
- [[Project_Structure]]
- [[Local_Dev_Setup]]
