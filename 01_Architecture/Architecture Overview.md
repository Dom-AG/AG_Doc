# Architecture Overview

tags: #architecture #decisions

## Stack

| Layer | Technology | Notes |
|---|---|---|
| Backend API | FastAPI + PostgreSQL | Async, OpenAPI docs auto-generated |
| ORM | SQLAlchemy + Alembic | Migrations as first-class work |
| Task Queue | Celery 5 + Redis 7 | Pipeline hooks, notifications |
| Desktop UI | PySide6 | Cross-platform, DCC integration |
| Environment Mgmt | Rez 3.x | ASWF standard |
| Auth | FastAPI-Users + JWT | Internal tool, no external dependency |

## Three-Layer Architecture

```
┌─────────────────────────────────────────┐
│           PySide6 Desktop App           │
│  (All views — Producer, Artist, Admin)  │
└──────────────────┬──────────────────────┘
                   │ HTTP / WebSocket (VPN)
┌──────────────────▼──────────────────────┐
│              FastAPI Backend            │
│        Business logic lives here        │
└──────────┬───────────────┬─────────────┘
           │               │
┌──────────▼──────┐ ┌──────▼──────────────┐
│   PostgreSQL    │ │    Celery + Redis    │
│  (source of     │ │  (async pipeline    │
│   truth)        │ │   hooks, notifs)    │
└─────────────────┘ └─────────────────────┘
```

## Component Responsibilities

**PySide6 App**
- All UI presentation
- Calls FastAPI — no direct DB access
- QSystemTrayIcon notifications
- WebSocket listener for live updates
- Rez launcher integration

**FastAPI**
- All business logic
- Auth + permissions
- Rate calculations
- Bid totals, variance reporting
- WebSocket broadcast on task events

**Celery**
- Pipeline hooks (render complete → update task)
- SyncSketch upload on review trigger
- Slack/email notifications on key events
- Weekly status report generation
- Nightly cost rollup jobs

**Rez**
- System-level tool, not a pipeline component
- Package definitions in `Packages/rez/`
- Python API used in-process — no subprocess shell wrapping

## Cross-Platform Notes

PySide6 runs on Linux, Windows, Mac from same codebase.

Paths must use `pathlib.Path` — never string concatenation.

NAS mount points differ per OS:

| OS | Mount |
|---|---|
| Linux | `/mnt/aslon/` |
| Mac | `/Volumes/aslon/` |
| Windows | `\\aslon\` or mapped drive |

Abstract behind config value — never hardcode.

## VPN / Connectivity

App communicates with FastAPI over HTTP — VPN is transparent to the application.

Every API call must handle `httpx.ConnectError` gracefully — show offline state, never crash.

Rez and local SQLite work offline. Task sync resumes when connection restores.

## Related

- [[Data Models]]
- [[Storage Strategy]]
- [[Rez Pipeline Integration]]
