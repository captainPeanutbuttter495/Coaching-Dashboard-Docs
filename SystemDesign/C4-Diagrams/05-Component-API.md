# C4 Model - Level 3b: Component Diagram (Insights API)

Inside `server/server.js`: middleware, three route groups, and the read pipeline every route shares. One file, 655 lines. The repo `README.md` carries a copy; change one, change both.

## Diagram

```mermaid
%%{init: {'theme': 'base', 'flowchart': {'useMaxWidth': true}, 'themeVariables': {'primaryColor': '#438DD5', 'primaryTextColor': '#fff', 'lineColor': '#666', 'fontSize': '14px'}}}%%
flowchart TD
    spa["<b>Web SPA</b>"]

    subgraph api ["Insights API · server/server.js"]
        mw["<b>Middleware</b><br/>CORS allowlist, express.json<br/>no auth"]
        roster["<b>Roster routes</b><br/>/api/getUsers, /api/updateRole"]
        single["<b>Per-athlete routes</b><br/>/api/heart-rate, sleep, vo2max<br/>+ compat aliases, /getVO2MaxData"]
        team["<b>Team routes</b><br/>/api/team/heart-rate, sleep,<br/>vo2max, respiratory-rate"]

        bts["<b>buildTeamSeries()</b><br/>sleep, vo2max, respiratory-rate<br/>serial"]
        fahr["<b>fetchAthleteHeartRows()</b><br/>heart-rate only, parallel"]
        norm["<b>toDate · clampAndSort · pickRows</b><br/>normalize, filter range, sort"]
        avg["<b>averageSeries · 60s bins</b><br/>mode=avg"]
        label["<b>getAthleteLabel()</b><br/>Auth name, Firestore name, uid"]
    end

    fs[("<b>Cloud Firestore</b><br/>HeartRateDataHC · SleepDataHC<br/>VO2MaxDataHX · RespiratoryRateDataHX")]
    auth["<b>Firebase Auth</b>"]

    spa -->|"JSON"| mw
    mw --> roster
    mw --> single
    mw --> team
    team --> bts
    team --> fahr
    bts --> norm
    fahr --> norm
    single --> norm
    bts --> avg
    fahr --> avg
    bts --> label

    norm -->|"full .get(), no .where()"| fs
    roster --> fs
    roster --> auth
    label --> auth
    fahr --> auth

    style spa fill:#438DD5,color:#fff,stroke:#3579B8
    style mw fill:#438DD5,color:#fff,stroke:#3579B8
    style roster fill:#438DD5,color:#fff,stroke:#3579B8
    style single fill:#438DD5,color:#fff,stroke:#3579B8
    style team fill:#438DD5,color:#fff,stroke:#3579B8
    style bts fill:#85BBF0,color:#000,stroke:#5D9FD6
    style fahr fill:#85BBF0,color:#000,stroke:#5D9FD6
    style norm fill:#85BBF0,color:#000,stroke:#5D9FD6
    style avg fill:#85BBF0,color:#000,stroke:#5D9FD6
    style label fill:#85BBF0,color:#000,stroke:#5D9FD6
    style fs fill:#999,color:#fff,stroke:#888
    style auth fill:#999,color:#fff,stroke:#888
    style api fill:#E8E8E8,color:#333,stroke:#BBB
```

### Legend

- **Blue** = middleware and route groups
- **Light blue** = module-scope helpers (all unexported)
- **Gray** = Firebase

## The Read Pipeline Every Route Shares

```
Firestore snapshot (the whole subcollection)
   ↓  clampAndSort(docs, startISO, endISO, pickRows)
   ↓     pickRows(doc)  per-metric field-name mapper, returns { ts, ...shaped }
   ↓     toDate(v)      Timestamp | { _seconds } | epoch ms | ISO string → Date
   ↓     filter to [start, end]
   ↓     sort ascending by ts
shaped rows → route-specific response shape
```

Two structural facts:

1. **Filtering happens in Node, not in Firestore.** Every route does a full `.get()` on the subcollection and discards documents outside the range afterwards. That exhausted the daily read quota on 2026-09-07. See [[API_Endpoints]].
2. **`pickRows` is where source inconsistency is absorbed.** Heart rate accepts `beatsPerMinute ?? bpm ?? y`; respiratory rate accepts `respiratoryRate ?? breathsPerMinute ?? rate ?? value ?? y`. [[Data_Model]] lists them all.

## Components

| Component | Responsibility |
|-----------|----------------|
| Credential load | `FIREBASE_SERVICE_ACCOUNT_JSON`, else `./firebase-service-account-prototype.json`, else exit naming both |
| CORS | `localhost:5173` + `FRONTEND_ORIGIN`; requests with no `Origin` pass |
| Roster routes | Resolve `authorizedAthletes` DocumentReferences one at a time; join Auth display names |
| Per-athlete routes | One subcollection, one athlete |
| Team routes | Many athletes; `overlay`/`grid` return `series`, `avg` returns `average` |
| `buildTeamSeries` | Shared by three team routes; serial; exact-timestamp averaging |
| `fetchAthleteHeartRows` | Heart-rate's own path; parallel; 60-second bins |
| `getAthleteLabel` | Auth `displayName` → Firestore name → uid |

## Key Relationships

| Relationship | Protocol | Purpose |
|-------------|----------|---------|
| SPA → API | HTTP JSON | Every health metric and roster |
| API → Firestore | firebase-admin, service account | Reads; bypasses security rules |
| API → Auth | firebase-admin | Display names; `getUserByEmail` for `updateRole` |

## Related Notes

- [[API_Endpoints]]
- [[03-Container]]
- [[06-Code]]
- [[Data_Model]]
