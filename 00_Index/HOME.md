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
- [[Pipeline Handover Checklist]]
- [[Rez Pipeline Integration]]
- [[Storage Strategy]]
- [[Services Node Plan]]
- [[Hosting]]
- [[Test Env]]
- [[OS Management]]

---

## Status

Current delivery status lives in [[Pipeline Handover Checklist]].

| Area | Status |
|---|---|
| REZ packaging | 🟢 Done |
| Relay database | 🟢 Done |
| Scoro API hooks to Relay | 🟢 Done |
| Schedule view for calendar | 🟡 Needs testing |
| Render Blades Deadline submitter | 🟡 Needs work |
| Relay allocations and task management | ⚪ Not started |

---

## Current Priorities

1. Test the schedule view and complete the Render Blades Deadline submitter.
2. Refine folder structure and connect DCC metadata/content data.
3. Set up production DCC environments and launcher project environments.

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
