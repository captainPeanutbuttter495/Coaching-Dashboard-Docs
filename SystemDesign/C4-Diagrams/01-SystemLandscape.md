# C4 Model - Level 0: System Landscape

The CSUN ARCS wearable-health ecosystem the Coaching Dashboard belongs to. The dashboard is a **read-and-render layer** over data another app writes.

- **Smart-Textile App** (`smart-clothing-app`): the athlete's mobile app. It syncs health data from the phone's health store into Firestore.
- **Coaching Dashboard** (this repo): reads that Firestore data and shows it to coaches.
- **Boracle Platform**: ARCS's standardization and consent middleware. The dashboard does **not** call it yet; that integration is planned.

## Diagram

```mermaid
%%{init: {'theme': 'base', 'flowchart': {'useMaxWidth': true}, 'themeVariables': {'primaryColor': '#438DD5', 'primaryTextColor': '#fff', 'lineColor': '#666', 'fontSize': '14px'}}}%%
flowchart TD
    athlete["<b>Athlete</b><br/>wears devices, owns data"]
    coach["<b>Coach</b><br/>reviews team trends"]

    subgraph arcs ["CSUN ARCS Ecosystem"]
        dash["<b>Coaching Dashboard</b><br/>coaching-dashboard<br/>React SPA + Express API"]
        mobile["<b>Smart-Textile App</b><br/>smart-clothing-app<br/>React Native / Expo"]
        boracle["<b>Boracle Platform</b><br/>boracle-platform<br/>Standardize, store, share"]
    end

    store["<b>Phone health stores</b><br/>Apple Health, Samsung Health"]
    wear["<b>Wearables</b><br/>watches, Hexoskin, smart textiles"]
    fb[("<b>Firebase</b><br/>Auth + Firestore")]

    athlete -->|"uses"| mobile
    athlete -->|"views own data"| dash
    coach -->|"uses"| dash
    wear -->|"vendor apps write to"| store
    mobile -->|"reads"| store
    mobile -->|"writes health time-series"| fb
    dash -->|"reads athlete data, auth"| fb
    dash -.->|"read standardized data (planned)"| boracle
    mobile -.->|"export data (planned)"| boracle

    style athlete fill:#08427B,color:#fff,stroke:#073B6F
    style coach fill:#08427B,color:#fff,stroke:#073B6F
    style dash fill:#1168BD,color:#fff,stroke:#0E5CA4
    style mobile fill:#999,color:#fff,stroke:#888
    style boracle fill:#999,color:#fff,stroke:#888
    style store fill:#999,color:#fff,stroke:#888
    style wear fill:#999,color:#fff,stroke:#888
    style fb fill:#999,color:#fff,stroke:#888
    style arcs fill:#E8E8E8,color:#333,stroke:#BBB
```

### Legend

- **Dark blue** = people
- **Blue (strong)** = the Coaching Dashboard, the system this vault documents
- **Gray** = sibling ARCS systems and external services (they exist)
- **Solid arrow** = code in the relevant repo makes this call today
- **Dashed arrow** = planned relationship

## Systems

| System | Repo | Stack | Relationship to the dashboard |
|--------|------|-------|-------------------------------|
| **Coaching Dashboard** | `CSU-Northridge-ARCS-Dev/coaching-dashboard` | React 18 + Vite (Netlify), Express + firebase-admin (Render) | — |
| **Smart-Textile App** | `CSU-Northridge-ARCS-Dev/smart-clothing-app` | React Native, Expo, Firebase | Writes every health document the dashboard reads. Not called by the dashboard |
| **Boracle Platform** | `CSU-Northridge-ARCS-Dev/boracle-platform` | Spring Boot 3, PostgreSQL, Firebase Auth | None in code. Documented in the Boracle-Docs vault |
| **Firebase** | — | Auth + Firestore | Auth for sign-in; Firestore is the system of record for users, invitations and health data |

## Why This Matters

- **The dashboard cannot fix data at the source.** Field names and timestamp types differ by writer, so the API normalizes on read. See [[Data_Model]].
- **Empty charts are often an upstream problem.** The newest sample in the project is 2025-03-27. Check that documents exist before debugging a chart.
- **Two ingestion paths exist in the ecosystem.** Boracle's Middleman App uploads to Boracle's PostgreSQL; the Smart-Textile App writes Firestore. The dashboard only sees the second.
- **Roles differ across systems.** Dashboard roles are `admin` / `coach` / `athlete` on `Users/{uid}`; Boracle's are `END_USER` / `DEVELOPER`. An integration needs a mapping.

> **Discrepancy with Boracle-Docs.** Its landscape draws "Coaching Dashboard → push invites via Expo → Smart-Textile App". No Expo or push-notification code exists in this repo on `refactor` or `main`. If that call exists, it lives in another repo or branch. Confirm before relying on either diagram.

## Related Notes

- [[02-SystemContext]]
- [[00-C4-Overview]]
- [[Data_Model]]
