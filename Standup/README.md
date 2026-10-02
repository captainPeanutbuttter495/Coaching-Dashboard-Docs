# Standup

Meeting transcripts, stand-up prep notes, supervisor question lists, and 1:1 notes for the Coaching Dashboard. Read the relevant ones before starting a ticket, because they hold the decisions and context behind the work.

---

## How This Folder Works

- **One file per meeting**, named by the meeting date so files sort chronologically:
  - `Standup_YYYY-MM-DD.md`: weekly stand-up (transcript or prep notes)
  - `Questions_YYYY-MM-DD[_Topic].md`: question lists for my supervisor or the previous team
  - `OneOnOne_YYYY-MM-DD_Topic.md`: 1:1 meetings
- Every file starts with frontmatter: `date`, `type`, `repo`, `ticket`, `source`. The `source` field points to the original file when one was imported.
- New meetings start from `_Templates/Standup_Template.md`. Paste the raw transcript under **Transcript / Notes**.
- **Claude's workflow:** when a ticket starts, search this folder for the ticket name and read the most recent matching notes. Answers here override assumptions.

---

## Index

| Date | File | Type | Ticket |
|------|------|------|--------|
| — | — | — | No meetings recorded yet |

### Open questions waiting for a meeting

These are recorded in `docs/refactor-phases.md` → "Open questions for the project owner" and have no answer yet:

1. Which system writes `Users/{uid}.authorizedAthletes`: the mobile app, or manual console edits? See [[Data_Model]].
2. Will the capstone team work against real athlete health data or seeded test data? That decides whether [[SEC_1_API_Auth_Tokens]] and [[SEC_2_Firestore_Rules]] must land first.
3. What does `HC` stand for in `HeartRateDataHC` / `SleepDataHC`? Probably Health Connect, but unconfirmed.

---

## Related Notes

- [[Ticket/README|Ticket index]]
- [[Sessions/README|Session log]]
- [[README|Vault home]]
