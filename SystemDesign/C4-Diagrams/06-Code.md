# C4 Model - Level 4: Code

The data and logic at code level: the Firestore schema the API reads, the role routing state machine in `ProtectedRoute`, and the `toDate()` normalization every time-series row passes through.

## 1. Firestore Schema

Observed, not enforced: the mobile app writes it. Field-level spellings and the full reasoning are in [[Data_Model]].

```mermaid
%%{init: {'theme': 'base', 'flowchart': {'useMaxWidth': true}, 'themeVariables': {'primaryColor': '#438DD5', 'primaryTextColor': '#fff', 'lineColor': '#666', 'fontSize': '14px'}}}%%
erDiagram
    USERS ||--o{ HEART_RATE_DATA_HC : "subcollection"
    USERS ||--o{ SLEEP_DATA_HC : "subcollection"
    USERS ||--o{ VO2MAX_DATA_HX : "subcollection"
    USERS ||--o{ RESPIRATORY_RATE_DATA_HX : "subcollection"
    USERS }o--o{ USERS : "authorizedAthletes refs"
    USERS ||--o{ INVITATIONS : "invitationsSent refs"

    USERS {
        string uid PK "Firebase Auth uid"
        string role "admin, coach or athlete"
        string email
        array authorizedAthletes "coach, DocumentReferences"
        array coachList "athlete, never written here"
        array pendingPermissions "athlete, written on invite"
        array invitationsSent "coach"
    }
    INVITATIONS {
        string id PK
        string coachId FK "plain uid string"
        string athleteEmail
        string status "pending"
    }
    HEART_RATE_DATA_HC {
        timestamp time "or timestamp, x"
        number beatsPerMinute "or bpm, y"
    }
    SLEEP_DATA_HC {
        timestamp startTime "or start"
        timestamp endTime "or end"
        number stage "1 Awake to 4 Deep"
    }
    VO2MAX_DATA_HX {
        timestamp timestamp
        number value "or vo2max, y"
    }
    RESPIRATORY_RATE_DATA_HX {
        timestamp time "or timestamp, date, x"
        number respiratoryRate "or breathsPerMinute, rate, value, y"
    }
```

### Legend

- One box per collection or subcollection; quoted text lists alternative spellings the API accepts
- Relationships are conventions: Firestore has no joins, foreign keys or cascades
- `HX` = Hexoskin (from a server comment); `HC` is unconfirmed

Literal paths:

```
Users/{uid}
Users/{uid}/HeartRateDataHC/{docId}          Users/{uid}/VO2MaxDataHX/{docId}
Users/{uid}/SleepDataHC/{docId}              Users/{uid}/RespiratoryRateDataHX/{docId}
Invitations/{invitationId}
```

## 2. Role Routing (`ProtectedRoute`)

```mermaid
%%{init: {'theme': 'base', 'flowchart': {'useMaxWidth': true}, 'themeVariables': {'primaryColor': '#438DD5', 'primaryTextColor': '#fff', 'lineColor': '#666', 'fontSize': '14px'}}}%%
stateDiagram-v2
    [*] --> AuthLoading
    AuthLoading --> SignedOut: no Firebase user
    AuthLoading --> RoleLoading: user present
    SignedOut --> [*]: Navigate to /
    RoleLoading --> NoRole: Users doc missing or no role
    RoleLoading --> Match: role equals requiredRole
    RoleLoading --> Mismatch: role differs
    NoRole --> NoRole: renders Loading role... forever
    Match --> [*]: render children
    Mismatch --> [*]: Navigate to /admin, /coach, /athlete or /
```

### Legend

- `AuthLoading` and `RoleLoading` render `Loading...` / `Loading role...`
- `NoRole` is a dead end: it is what the `matthew@gmail.com` test account hits
- Passing this gate does **not** mean the page renders: `<AllowedAccess>` is a second, independent gate. See [[Auth_And_Roles]]

## 3. Timestamp Normalization (`toDate`)

```mermaid
%%{init: {'theme': 'base', 'flowchart': {'useMaxWidth': true}, 'themeVariables': {'primaryColor': '#438DD5', 'primaryTextColor': '#fff', 'lineColor': '#666', 'fontSize': '14px'}}}%%
flowchart TD
    v["<b>raw field value</b>"]
    t1["<b>Firestore Timestamp</b><br/>proper write"]
    t2["<b>object with _seconds</b><br/>serialized Timestamp"]
    t3["<b>number</b><br/>epoch milliseconds"]
    t4["<b>ISO string</b><br/>written as text"]
    ok["<b>JS Date</b><br/>kept, filtered to range, sorted"]
    bad["<b>null</b><br/>row dropped silently"]

    v --> t1 --> ok
    v --> t2 --> ok
    v --> t3 --> ok
    v --> t4 --> ok
    v -->|"anything else,<br/>or unparseable"| bad

    style v fill:#08427B,color:#fff,stroke:#073B6F
    style t1 fill:#438DD5,color:#fff,stroke:#3579B8
    style t2 fill:#438DD5,color:#fff,stroke:#3579B8
    style t3 fill:#438DD5,color:#fff,stroke:#3579B8
    style t4 fill:#438DD5,color:#fff,stroke:#3579B8
    style ok fill:#1168BD,color:#fff,stroke:#0E5CA4
    style bad fill:#C94C4C,color:#fff,stroke:#8D3535
```

### Legend

- **Blue** = the four accepted input types
- **Red** = the silent drop: no log, just fewer chart points

Sleep rows are intervals: they are kept if they **intersect** the requested window, not only if they fall inside it.

## Key Relationships

| Relationship | Where | Purpose |
|-------------|-------|---------|
| `authorizedAthletes` → `Users` | coach doc | The roster `/api/getUsers` reads, one `get()` per reference |
| `Invitations.coachId` → `Users` | plain string | Queried with `where("coachId", "==", …)` |
| `toDate` ← every `pickRows` | `server/server.js` | One timestamp funnel for all metrics |

## Related Notes

- [[Data_Model]]
- [[05-Component-API]]
- [[Auth_And_Roles]]
- [[07-Dynamic-Diagrams]]
