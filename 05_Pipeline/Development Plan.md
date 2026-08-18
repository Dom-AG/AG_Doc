# Development Plan

tags: #planning #roadmap #pipeline

> Living document. Update sprint status as work progresses.
> All schema changes require Alembic migration + written rationale before touching production.

---

## Guiding Principles

- **Each phase delivers something usable** — no long dark periods where nothing works
- **Boring technology** — resist new frameworks unless existing cannot solve the problem
- **Written RFC** before any feature outside this plan gets added
- **Delete unused features** — do not comment out, do not defer, delete
- **One error at a time** — fix before moving forward, never accumulate broken state

---

## Stack Reference

| Layer         | Technology                                  |
| ------------- | ------------------------------------------- |
| Backend       | FastAPI + PostgreSQL + SQLAlchemy + Alembic |
| Task Queue    | Celery 5 + Redis 7                          |
| Desktop UI    | PySide6                                     |
| Environment   | Rez 3.x (system tool)                       |
| HTTP Client   | httpx (async)                               |
| Auth          | FastAPI-Users + JWT                         |
| Notifications | QSystemTrayIcon + WebSockets                |
| Packaging     | PyInstaller (per platform)                  |

---

## Phase Overview

| Phase | Focus | Weeks | Risk |
|---|---|---|---|
| 1 | Foundation — API + DB + Rez | 1–6 | Low |
| 2 | People, Skills, Rates + Bid Form | 5–12 | Medium |
| 3 | Producer Calendar (Scheduler) | 10–16 | Medium |
| 4 | Project View + Artist Kanban | 14–20 | Low |
| 5 | Pipeline Hooks + Rez Launcher | 16–22 | Medium |
| 6 | Reporting, Notifications, Polish | 20–26 | Low |

Phases overlap deliberately — later phases can begin once earlier phase is stable, not necessarily fully complete.

---

## Phase 1 — Foundation (Weeks 1–6)

**Goal:** Running API, working database, one Rez package resolving end to end.

### 1.1 Rez Setup

- [ ] Write `rezconfig.py` with correct NAS paths
  - `local_packages_path = "~/.rez/packages"`
  - `release_packages_path = "/mnt/aslon/04_Tools/REZ/packages"`
- [ ] Write `houdini/20.5.584/package.py` pointing at `/opt/hfs20.5.584`
- [ ] Fix `Launcher/plugins/environment.py` — replace subprocess with Python API
  ```python
  from rez.resolved_context import ResolvedContext
  ctx = ResolvedContext(["houdini-20.5.584"])
  env = ctx.get_environ()
  ```
- [ ] Verify: `rez-env houdini-20.5.584 -- houdini` launches correctly
- [ ] Confirm NAS automounts on startup (`/etc/fstab` or equivalent)

### 1.2 PostgreSQL + FastAPI Skeleton

- [ ] Docker Compose dev environment (PostgreSQL + Redis + FastAPI)
- [ ] SQLAlchemy async engine + session factory
- [ ] Alembic configured — first migration creates all tables
- [ ] FastAPI app structure:
  ```
  API/
    main.py
    database.py
    models.py       # SQLAlchemy ORM
    schemas.py      # Pydantic request/response
    routers/
      projects.py
      clients.py
      people.py
      skills.py
      shots.py
      assets.py
  ```
- [ ] JWT auth via FastAPI-Users — login endpoint, protected routes
- [ ] Health check endpoint (`GET /health`) for VPN connectivity check

### 1.3 Core Schema (First Migration)

Tables in first Alembic migration:

- [ ] `clients` — id, name, currency, notes
- [ ] `projects` — id, name, code, client_id, status, start_date, end_date, root_path, base_packages (JSONB), dcc_envs (JSONB)
- [ ] `people` — id, name, email, department, active, default_currency
- [ ] `skills` — id, name, department
- [ ] `person_skills` — person_id, skill_id, day_rate, overtime_rate, currency, effective_from, effective_to
- [ ] `shots` — id, project_id, name, code, status, cut_in, cut_out, complexity, department
- [ ] `tasks` — id, shot_id, skill_id, assigned_to, status, due_date, started_at, stopped_at, priority, notes
- [ ] `time_entries` — id, person_id, task_id, date, started_at, stopped_at, hours, source

### 1.4 SQLite Migration

- [ ] Migrate existing SQLite data via pgloader
- [ ] Verify all existing projects/assets present in PostgreSQL
- [ ] Keep SQLite provider for offline dev — `DatabaseProvider` abstract class intact

**Deliverable:** `docker compose up` works, API returns data, Houdini launches via Rez Python API.

---

## Phase 2 — People, Skills, Rates + Bid Form (Weeks 5–12)

**Goal:** Producers can create and send a bid. Financial foundation established.

### 2.1 People + Skills CRUD

- [ ] FastAPI endpoints: CRUD for People, Skills, PersonSkills
- [ ] Rate history logic — closing old rate when new one added
- [ ] PySide6 People management screen:
  - List view filterable by department / active status
  - Click person → side panel with skills and rates
  - Add/edit skill rate — creates new PersonSkill row, closes previous
  - Deactivate person — sets `active = false`

### 2.2 Bid Form Schema (Second Migration)

- [ ] `bid_forms` — id, project_id, version, status, created_by, created_at, notes, currency
- [ ] `bid_lines` — id, bid_form_id, skill_id, days_estimated, rate_snapshot, currency_snapshot, phase, department, notes

### 2.3 Bid Form UI

- [ ] Header section — project, client, version, status badge, currency, notes
- [ ] Line items table — inline editing, add/delete rows
  - Skill dropdown → auto-fills rate from active PersonSkill
  - Rate manually overridable
  - Line total auto-calculates (days × rate)
- [ ] Totals panel — subtotal per department, subtotal per phase, grand total
- [ ] Multi-currency warning if mixed currencies
- [ ] Status transitions — Draft → Sent (freezes rate snapshots) → Approved / Rejected

### 2.4 Bid Versioning

- [ ] New version creates copy of all BidLines from previous version
- [ ] Previous versions read-only
- [ ] Version selector in bid form header

### 2.5 PDF Export

- [ ] FastAPI endpoint generates PDF via ReportLab or WeasyPrint
- [ ] PySide6 triggers download and opens file
- [ ] PDF includes: studio header, client/project, version/date, line items, totals, assumptions

**Deliverable:** Producer can create, version, and export a bid. Rate history preserved. Financial baseline established.

---

## Phase 3 — Producer Calendar / Scheduler (Weeks 10–16)

**Goal:** Xytech-style view — people as rows, days as columns, drag to allocate.

### 3.1 Allocation Schema (Third Migration)

- [ ] `allocations` — id, person_id, skill_id, project_id, bid_line_id (nullable), start_date, end_date, days_allocated, notes
- [ ] `studio_holidays` — id, date, name (for capacity calculation)

### 3.2 Scheduler API Endpoints

- [ ] `GET /allocations?start=&end=&department=` — range query for calendar view
- [ ] `POST /allocations` — create allocation
- [ ] `PATCH /allocations/{id}` — update dates/days (drag result)
- [ ] `DELETE /allocations/{id}`
- [ ] `GET /people/{id}/availability?start=&end=` — capacity check

### 3.3 Scheduler UI (PySide6 Custom Widget)

- [ ] Custom `QWidget` with `paintEvent` — canvas-based timeline
- [ ] Rows = people (grouped by department, collapsible)
- [ ] Columns = days (zoom: day / week / month / quarter)
- [ ] Bars = allocations, coloured per project
- [ ] Drag bar → move allocation dates (`mousePressEvent`, `mouseMoveEvent`, `mouseReleaseEvent`)
- [ ] Drag bar edge → resize allocation
- [ ] Drag from unassigned row → person row = create new allocation
- [ ] Click bar → detail popover
- [ ] Double-click bar → open project panel

### 3.4 Capacity Indicators

- [ ] Colour per person row: green (<80%) / amber (80–100%) / red (>100%)
- [ ] Calculation: `sum(allocation.days) / available_working_days` excluding weekends + holidays
- [ ] Conflict detection: red outline on bar if person double-booked same dates

### 3.5 Filters

- [ ] Filter by department, project, person
- [ ] Group by department (default) / project / skill
- [ ] Zoom level toggle — remembers last used per user

**Deliverable:** Producers can visually schedule people across projects, spot conflicts, and link allocations back to bid lines.

---

## Phase 4 — Project View + Artist Kanban (Weeks 14–20)

**Goal:** Gantt view for supervisors/producers. Kanban + time tracking for artists.

### 4.1 Project View (Gantt)

- [ ] Custom `QWidget` paintEvent — similar approach to scheduler
- [ ] Top level: project bars (full duration)
- [ ] Double-click project → expands to shot rows
- [ ] Double-click shot → expands to task rows with assignee label
- [ ] Drag bar / edge → update shot/task dates
- [ ] Status colours per bar (pending grey / in-progress amber / review blue / approved green / blocked red)

### 4.2 Artist Kanban

- [ ] Columns: Pending / In Progress / Review / Done
- [ ] Show only tasks assigned to logged-in user, due today or overdue
- [ ] Cards: shot code, task type, project name, planned hours, elapsed timer
- [ ] Drag Pending → In Progress: `POST /time-entries` with `started_at = now()`
- [ ] Drag In Progress → Pending: PATCH time entry `stopped_at = now()` (pause)
- [ ] Drag In Progress → Review: PATCH time entry `stopped_at = now()` (complete)
- [ ] Pipeline trigger → Review: same endpoint, `source = "pipeline_trigger"`

### 4.3 Daily Hour Planning

- [ ] `daily_allocations` table — person_id, task_id, date, planned_hours
- [ ] Artist sets planned hours per card at start of day
- [ ] Card shows planned vs actual elapsed

### 4.4 Task Side Panel

- [ ] Double-click card → panel slides in from right
- [ ] Full task details, status history, time entries
- [ ] Comments thread — `task_comments` table (id, task_id, person_id, body, created_at)
- [ ] Pipeline events log
- [ ] Linked asset versions
- [ ] SyncSketch review link if in Review status

**Deliverable:** Artists track their day in Kanban. Actuals feed variance reporting. Supervisors see shot progress in Gantt.

---

## Phase 5 — Pipeline Hooks + Rez Launcher (Weeks 16–22)

**Goal:** Pipeline events update task state automatically. DCC launches with correct Rez environment.

### 5.1 Celery Workers

- [ ] Celery + Redis configured in Docker Compose
- [ ] Worker for: task status change → Slack notification
- [ ] Worker for: task → Review → SyncSketch upload
- [ ] Worker for: render complete → task status update + artist notification
- [ ] Celery Beat: nightly cost rollup (TimeEntry hours × rates → project actuals)
- [ ] Celery Beat: weekly status report generation

### 5.2 Rez Launcher (full)

- [ ] `ag_core` Rez package — shared pipeline Python
- [ ] `ag_houdini` Rez package — HDAs + pipeline scripts, points to NAS
- [ ] `ag_maya` Rez package
- [ ] `ocio` Rez package — sets `OCIO` env var to ACES config on NAS
- [ ] PySide6 standalone launcher UI:
  - Project picker (queries FastAPI)
  - DCC selector per project
  - Resolves Rez env via Python API
  - Launches DCC with merged environment
- [ ] `PostgresProvider` — Launcher connects to FastAPI via httpx

### 5.3 Houdini Pipeline Panel

- [ ] Thin Python Panel — status display + task switcher only
- [ ] Queries FastAPI for current artist's tasks
- [ ] Publish button → creates AssetVersion via API
- [ ] No business logic in panel — all calls go through FastAPI

### 5.4 WebSockets

- [ ] FastAPI WebSocket endpoint — broadcasts task events
- [ ] PySide6 `QWebSocket` connection on app startup
- [ ] On event received: update relevant view + show tray notification

**Deliverable:** Pipeline events flow automatically. DCC launches in correct environment. Artists notified without manual refresh.

---

## Phase 6 — Reporting, Notifications, Polish (Weeks 20–26)

**Goal:** Variance reporting. Production-ready stability. Cross-platform packaging.

### 6.1 Financial Reporting

- [ ] Bid vs actuals variance report per project
- [ ] Department breakdown — days bid vs days actual vs days remaining
- [ ] Person utilisation report — allocated vs actual per period
- [ ] Export to CSV / PDF from FastAPI endpoint
- [ ] ⚠️ Xero integration scope: confirm with Sara before building

### 6.2 Asset Version Browser

- [ ] Asset version list per shot — queryable from project view
- [ ] Version lineage visualisation (VersionDependency table)
- [ ] Publish event from Houdini creates AssetVersion via API
- [ ] SyncSketch review link attached to version on upload

### 6.3 Notifications (full)

- [ ] `QSystemTrayIcon` tray notifications wired to WebSocket events
- [ ] Notification types per event table (see [[Views and Notifications]])
- [ ] Celery → Slack on high-priority events (blocked shots, overdue tasks)

### 6.4 Cross-Platform Packaging

- [ ] PyInstaller build script — separate builds for Linux, Windows, Mac
- [ ] `.env` config for API URL — single change per studio machine
- [ ] Mac: internal use only — skip notarization for now
- [ ] Test on all three platforms before declaring stable

### 6.5 AI Integration (additive, never blocking)

- [ ] Brief/email summarisation → auto-populate task fields (Anthropic API)
- [ ] Whisper transcription → meeting notes → action items
- [ ] Weekly status report draft via Celery Beat
- [ ] UI panel: paste brief → review AI-extracted data → confirm

**Deliverable:** Full production-ready system. Reporting visible. Packaged for all studio machines.

---

## Open Questions (Resolve Before Building)

| Question | Owner | Needed For |
|---|---|---|
| Bid in days or hours? | Charlotte | Phase 2 |
| Overtime rate on bid lines — include in v1? | Charlotte | Phase 2 |
| VAT line item on PDF export? | Charlotte / Accountant | Phase 2 |
| Multi-currency per bid line or per bid? | Charlotte | Phase 2 |
| Who can create/edit bids? | Charlotte | Phase 2 |
| Contractor vs staff — same rate model? | Charlotte | Phase 2 |
| Rate visibility — can artists see own rate? | Domantas | Phase 2 |
| Part-day allocations (50/50 split)? | Charlotte | Phase 3 |
| Holiday calendar — studio-wide or per person? | Charlotte | Phase 3 |
| Xero integration scope | Accountant | Phase 6 |
| Hours per day constant (8 or 10)? | Charlotte | Phase 2 |

---

## Anti-Patterns to Avoid

- ❌ Custom video annotation player — use SyncSketch
- ❌ Blender integration — companion app only, out of scope
- ❌ `rez-env` subprocess wrapping — use Python API
- ❌ Business logic in PySide6 widgets — thin UI, logic in FastAPI
- ❌ Direct DB access from Launcher — always through FastAPI
- ❌ Storing binary data in PostgreSQL — paths only
- ❌ Hard-coded NAS paths — always from config
- ❌ Mixed SQLite + PostgreSQL sync — PostgreSQL is source of truth

---

## Related

- [[Architecture Overview]]
- [[Data Models]]
- [[Bid Form Spec]]
- [[Producer Calendar View]]
- [[Rez Pipeline Integration]]
- [[Views and Notifications]]
- [[People and Skills]]
