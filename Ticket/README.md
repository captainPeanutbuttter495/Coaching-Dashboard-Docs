# Ticket

A running history of **my** work on the Coaching Dashboard: one note per ticket, refactor phase or PR, each ending with resume-ready bullets. [[Resume_Highlights]] collects the best of them. For what the rest of the team shipped, see [[Project_History]].

---

## How This Folder Works

- New tickets start from `_Templates/Ticket_Template.md`. Name them `Ticket_Name.md`, PascalCase with underscores. A name must not match a note in another folder, because wiki-links resolve by file name. For example, the API note is `API_Endpoints`, so the auth ticket is `SEC_1_API_Auth_Tokens`.
- Frontmatter fields:
  - `status`: `planned` · `in-progress` · `in-review` · `committed` (on `refactor`, not yet on `main`) · `merged` · `blocked` · `needs-confirmation`
  - `repo`, `area`, `branch`, `pr`, `commit`
  - dates, `standups`, `skills`
- **Resume bullets** start with an action verb and use only numbers that are actually recorded (in `docs/refactor-phases.md`, git, or a test run). Never invent metrics.
- When a ticket lands, update its status and bullets, then refresh [[Resume_Highlights]].
- The repo also has `tickets/<ID>/ticket.md` for the four-pane ticket pipeline (`docs/ai-orchestration.md`). Those are the working scope; the note here is the write-up. Link them by ID.
- Author on GitHub: `captainPeanutbuttter495` (and the earlier `captain.peanutbutter495`).

---

## Index

| # | Ticket | Area | Commit / PR | Dates | Status |
|---|--------|------|-------------|-------|--------|
| 1 | [[Early_Frontend_Contributions]] | frontend | 10 commits | 2024-03-24 → 2025-03-10 | merged |
| 2 | [[Readme_And_Documentation_Vault]] | docs | #32 (`21fc4e8`) | 2026-09-03 | merged |
| 3 | [[SEC_1_API_Auth_Tokens]] | api | — | planned 2026-09-03 | planned |
| 4 | [[SEC_2_Firestore_Rules]] | api | — | planned 2026-09-03 | planned |
| 5 | [[Refactor_Phase_1_Dead_Code]] | refactor | `b539ba3` | 2026-09-05 | committed |
| 6 | [[Refactor_Phase_2_Vite_Layout]] | refactor | `45f4258` | 2026-09-05 | committed |
| 7 | [[Refactor_Phase_3_Features]] | refactor | `4fa4ca3`, `d7443a9` | 09-06 → 09-07 | committed |
| 8 | [[Refactor_Phase_4_Dashboard_Data]] | refactor | `28a8a25` | 2026-09-07 | committed |
| 9 | [[Refactor_Phase_5_Styling_Pass]] | refactor | `d9c5d92` | 2026-09-07 | committed |
| 10 | [[Refactor_Phase_8A_Team_Insights]] | refactor | `d9c5d92` | 2026-09-07 | committed |
| 11 | [[Refactor_Phase_8B_Dashboard_Shell]] | refactor | `1374879` | 2026-09-07 | committed |
| 12 | [[Refactor_Phase_8C_Team_Builder]] | refactor | `5ae5ae6` | 2026-09-14 | committed |
| 13 | [[Refactor_Phase_8D_Inline_Styles]] | refactor | `5ae5ae6` | 2026-09-14 | committed |
| 14 | [[Refactor_Phase_6_Tooling]] | tooling | `99a9eb6` | 2026-09-14 | committed |
| 15 | [[Vault_Restructure]] | docs | — (vault is gitignored) | 2026-09-29 | in review |

The refactor branch is 9 commits ahead of `origin/main @ 666ede1` and has not been merged. Phase 7 (documentation refresh) is not started; Phase 0 (branch + baseline) has no separate note.

Sources: `git log --author=captain`, `docs/refactor-phases.md`, `tickets/SEC-*/ticket.md`.

---

## Related Notes

- [[Resume_Highlights]]
- [[Project_History]]
- [[Refactor]]
- [[Standup/README|Standup index]]
- [[Sessions/README|Session log]]
