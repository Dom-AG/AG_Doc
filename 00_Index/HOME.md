# AG-Pipeline Vault

> Internal spec documentation for the AG-Pipeline production management system.

## Quick Links

- [[Architecture Overview]]
- [[Data Models]]
- [[Bid Form Spec]]
- [[Producer Calendar View]]
- [[Project View]]
- [[Artist Kanban View]]
- [[Notifications]]
- [[Rez Pipeline Integration]]
- [[Storage Strategy]]
- [[Services Node Plan]]
- [[Hosting]]
- [[Test Env]]
- [[OS Management]]

---

## Status

| Area | Status | Owner |
|---|---|---|
| Architecture | ✅ Decided | Domantas |
| Data Models | ✅ Decided | Domantas |
| Bid Form | 🔄 Speccing | Domantas + Charlotte |
| Producer Calendar | 🔄 Speccing | Domantas + Charlotte |
| Artist Kanban | 📋 Planned | Domantas |
| Project View | 📋 Planned | Domantas |
| Rez Integration | 🔄 In Progress | Domantas |
| UI Style | ✅ Decided | Domantas |

---

## Build Phases

1. **Phase 1** — [[Bid Form Spec]] + People/Skills/Rates
2. **Phase 2** — [[Producer Calendar View]] (Xytech-style scheduler)
3. **Phase 3** — [[Project View]] (Gantt, shot-level)
4. **Phase 4** — [[Artist Kanban View]] + time tracking
5. **Phase 5** — AI integration + reporting
6. **Phase 6** — Asset versioning + review pipeline

---

## Key Decisions Log

- Bid in **days**, track actuals in **hours** — confirm `hours_per_day` constant with Charlotte
- Rates are **per person per skill** — not global skill rates
- Rates **frozen at bid time** — historical bids never recalculate
- UI stack: **PySide6 + FastAPI** — single language, DCC integration, VPN-transparent
- Database: **PostgreSQL** via FastAPI — SQLite for local/offline dev only
- No web frontend — PySide6 handles all views including producer/management
- Notifications via **QSystemTrayIcon + WebSockets**
- Video review: **SyncSketch** — no custom annotation player
