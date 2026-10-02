# Sessions

A dated log of work sessions on the Coaching Dashboard: what changed, what was decided and why, and what was left open. Written from `git status` / `git diff`, not from memory.

---

## How This Folder Works

- **One file per session**, named `Session_YYYY-MM-DD_Short_Slug.md` so files sort chronologically. If a second, unrelated session happens the same day, give it its own slug; related work extends the day's note under `## Later That Day`.
- New notes start from `_Templates/Session_Template.md`. Frontmatter: `title`, `date`, `branch`, `head`, `tickets`, `tags: [session]`.
- **The usual way to write one is `/session-log`** (`.claude/commands/session-log.md`), run by the owner at the end of a session. It reads `git status --porcelain` and `git diff`, writes the note here, fixes any domain note the session made wrong, updates the matching `Ticket/` note, and adds a row to the index below. It never touches git state: the note lands in the working tree next to the code it describes.
- Doing it by hand is fine. Copy the template, fill it in, add the index row. The convention matters; the tooling does not.

### Session log vs. domain notes

A session note records *what happened on one day*. The notes in `Project/`, `Frontend/`, `Backend/`, `Infrastructure/` and `SystemDesign/` record *what is true now*. If a change makes one of those wrong, **fix that note**; do not leave the correction buried in a dated log where nobody will find it. The session note then links to the note it corrected.

Rule of thumb: **if someone reading only the domain notes would now be misled, the fix belongs there.**

### Session vs. ticket

A **ticket** note (`Ticket/`) is one unit of shipped work, with resume bullets, and may span several sessions. A **session** note is one sitting, and may touch several tickets. Link them both ways: `tickets:` in the session's frontmatter, and the session under the ticket's Related Notes.

---

## Index

| Date | Session | Branch | Tickets |
|------|---------|--------|---------|
| 2026-09-03 | [[Session_2026-09-03_Readme_And_Vault]] | `docs/readmeUpdate` | [[Readme_And_Documentation_Vault]] |

The refactor phases (2026-09-05 → 09-14) were each one session, but were recorded in `docs/refactor-phases.md` rather than here. Their write-ups are the `Refactor_Phase_*` notes in [[Ticket/README|Ticket]].

---

## Related Notes

- [[Ticket/README|Ticket index]]
- [[Standup/README|Standup index]]
- [[README|Vault home]]
