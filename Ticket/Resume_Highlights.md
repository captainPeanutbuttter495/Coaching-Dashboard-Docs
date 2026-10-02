---
role: "TODO confirm exact title — CSUN ARCS, Coaching Dashboard"
period: "2024-03 → present"
updated: 2026-09-29
tags: [resume]
---

# Resume Highlights

The strongest bullets from the [[Ticket/README|ticket history]], deduplicated and grouped by skill. Pick 3–5 per application, depending on the role. Every number here is recorded in `docs/refactor-phases.md` or git.

---

## Summary Line

Developer on CSUN ARCS's Coaching Dashboard, a React + Express + Firebase app that turns athlete wearable data into coaching insights. I built its role-based access and first health charts, then led a measured, zero-regression refactor to prepare it for handoff to a capstone team.

---

## Architecture / Refactoring

- Restructured a React SPA into a feature-oriented architecture (47 files moved, 32 import sites rewritten to a path alias) with a byte-identical build, as the foundation for a capstone team handoff. → [[Refactor_Phase_3_Features]]
- Audited and removed dead code before a handoff: 10 orphan files, 20 unused packages and ~1,100 lines of commented-out code, cutting installed packages from 989 to 547 with no behaviour change. → [[Refactor_Phase_1_Dead_Code]]
- Separated mock data from presentation across 12 dashboard cards, adding prop-based injection seams for a future API, verified byte-identical with server-side rendering comparisons. → [[Refactor_Phase_4_Dashboard_Data]]

## Build / Deploy

- Diagnosed and fixed three production deploy bugs traced to one non-standard Vite setting: a missing SPA fallback on Netlify, a Windows/Linux case-sensitivity build difference, and implicit Firebase initialization. → [[Refactor_Phase_2_Vite_Layout]]
- Identified a git case-sensitivity hazard on Windows that would have broken the Linux production build. → [[Refactor_Phase_3_Features]]

## CSS / Tailwind

- Completed a full migration from a 3,398-line hand-written stylesheet to Tailwind CSS across four phases with zero measured rendering differences. → [[Refactor_Phase_8C_Team_Builder]]
- Built a codemod that converted 162 inline React style objects to Tailwind classes, returning what it could not convert faithfully as residue; 115/115 identical computed-style captures at every step. → [[Refactor_Phase_8D_Inline_Styles]]
- Found and fixed a silent Tailwind configuration failure (object-form `screens` disabling all `max-*` variants) by building the CSS under controlled variations and diffing the output. → [[Refactor_Phase_8A_Team_Insights]]

## Testing / Verification

- Built a visual-regression method for a CSS refactor: full-page computed-style dumps across five viewports in an automated browser, with a measured zero noise floor. It caught regressions that visual review missed. → [[Refactor_Phase_5_Styling_Pass]], [[Refactor_Phase_8A_Team_Insights]]
- Verified hover and pseudo-state styling programmatically through the Chrome DevTools Protocol (7/7, then 28/28 states matching the deleted CSS). → [[Refactor_Phase_8B_Dashboard_Shell]], [[Refactor_Phase_8C_Team_Builder]]
- Quantified the unreliability of screenshot testing on chart-heavy pages (30 of 105 captures differ on identical code) and moved the project's evidence to computed-style dumps. → [[Refactor_Phase_8A_Team_Insights]]

## Tooling / Developer Experience

- Introduced ESLint and Prettier to an unlinted React/Express codebase, reaching 0 errors by fixing 75 findings mechanically and proving the shipped bundle unchanged by diffing it. → [[Refactor_Phase_6_Tooling]]
- Encoded an architecture rule as lint errors (import direction between layers, case-sensitive paths) and verified each rule with negative tests. → [[Refactor_Phase_6_Tooling]]

## Frontend Features

- Built role-based navigation and access (admin / coach / athlete) for a React + Firebase dashboard, including the login flow, redirects and an admin user grid. → [[Early_Frontend_Contributions]]
- Implemented the first athlete health charts (sleep stages, heart rate, activity ring) and connected them to live Firestore data. → [[Early_Frontend_Contributions]]

## Security / Debugging

- Surfaced an unauthenticated API exposing athlete health data during a documentation audit, and scoped the fix as a security ticket. → [[Readme_And_Documentation_Vault]], [[SEC_1_API_Auth_Tokens]]
- Diagnosed a Firestore outage (daily read quota exhausted by unbounded queries) that initially presented as a CSS regression. → [[Refactor_Phase_8B_Dashboard_Shell]]

## Documentation

- Wrote architecture documentation with C4 diagrams for a three-service React/Express/Firebase application, verifying every claim against the source. → [[Readme_And_Documentation_Vault]]
- Standardized documentation across three projects into one vault structure with C4 diagrams, ticket histories and a sync workflow. → [[Vault_Restructure]]

---

## Tech Keywords

React 18 · Vite · Tailwind CSS · JavaScript (ES modules) · Node.js · Express · Firebase Authentication · Cloud Firestore · firebase-admin · Chart.js · Recharts · MUI · ESLint · Prettier · Vitest · Chrome DevTools Protocol · Netlify · Render · C4 model · Mermaid
