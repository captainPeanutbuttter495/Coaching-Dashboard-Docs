# System Design

Architecture documentation for the Coaching Dashboard: C4 model diagrams covering the SPA, the Insights API and the email service, plus where the dashboard sits in the CSUN ARCS ecosystem.

---

## C4 Diagrams

The `C4-Diagrams/` subfolder follows Simon Brown's [C4 model](https://c4model.com/) (Context, Containers, Components, Code).

| Level | File | Description |
|-------|------|-------------|
| — | [00-C4-Overview](C4-Diagrams/00-C4-Overview.md) | Index: hierarchy, palette, key decisions |
| **0** | [01-SystemLandscape](C4-Diagrams/01-SystemLandscape.md) | ARCS ecosystem: dashboard, Smart-Textile app, Boracle, Firebase |
| **1** | [02-SystemContext](C4-Diagrams/02-SystemContext.md) | The dashboard, its three roles, external systems |
| **2** | [03-Container](C4-Diagrams/03-Container.md) | SPA, Insights API, email service; the two paths to Firebase; dev vs prod |
| **3a** | [04-Component-Frontend](C4-Diagrams/04-Component-Frontend.md) | Router, gates, pages, feature folders, `lib/` |
| **3b** | [05-Component-API](C4-Diagrams/05-Component-API.md) | Middleware, route groups, the shared read pipeline |
| **4** | [06-Code](C4-Diagrams/06-Code.md) | Firestore ER, role routing states, `toDate()` |
| **Dyn** | [07-Dynamic-Diagrams](C4-Diagrams/07-Dynamic-Diagrams.md) | Login + refresh bug, Team Insights fetch, invitation |

All diagrams are **Mermaid** and render natively in Obsidian. `.obsidian/snippets/mermaid-fit.css` scales them to width.

---

## README Duplicates

The repo's `README.md` carries copies of the context, container and component diagrams: the one duplication this vault accepts. Change one, change both.

> As of `refactor @ 99a9eb6` the README's copies and its project tree are **pre-refactor** (they show `root: "src"`, `Pages/`, `Components/`, `firebaseConfig.js`). Refreshing them is refactor Phase 7's job. The diagrams here are current.

---

## Related Notes

- [[Tech_Stack]]
- [[Project_Structure]]
- [[Roadmap]]
- [[Known_Issues]]
