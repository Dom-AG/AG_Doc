# Project View

tags: #feature #priority-3 #producer #supervisor

> High-level Gantt. Projects → Shots → Tasks expanding downward.

---

## Layout

```
┌──────────────────────────────────────────────────────────────┐
│ Project Alpha          ████████████████████████████████████  │
│   ▼ Shot_010           ████████                              │
│       Comp Task        ████████  Alice                       │
│       FX Task              ████████  Bob                     │
│   ▼ Shot_020                    ████████████                 │
│       Comp Task                 ████████  Alice              │
│ Project Beta           ░░░░░░░████████████████               │
└──────────────────────────────────────────────────────────────┘
```

- Top level: Projects (single bar, full duration)
- Double-click project bar → expands to shots
- Double-click shot → expands to tasks with assignee shown
- Drag bar → move shot/task dates
- Drag bar edge → extend or shorten

## Status Colours

| Status | Colour |
|---|---|
| Pending | Grey |
| In Progress | Amber |
| Review | Blue |
| Approved | Green |
| Blocked | Red |
| Omit | Strikethrough |

## Related
- [[Producer Calendar View]]
- [[Data Models]]

---

# Artist Kanban View

tags: #feature #priority-4 #artist #time-tracking

> Personal board. Today's tasks only. Drag triggers time tracking.

---

## Columns

```
[ PENDING ]     [ IN PROGRESS ]     [ REVIEW ]     [ DONE ]
  Card              Card               Card           Card
  Card                                 Card
```

Only tasks assigned to logged-in user, due today or overdue.

## Card Content
- Shot code + task type
- Project name (colour coded)
- Estimated hours (set by artist at start of day)
- Elapsed time (live timer if In Progress)
- Status indicator dot

## Drag Interactions

| Action | System records |
|---|---|
| Pending → In Progress | `TimeEntry.started_at = now()` |
| In Progress → Pending | `TimeEntry.stopped_at = now()` (pause) |
| In Progress → Review | `TimeEntry.stopped_at = now()` (work complete) |
| Pipeline trigger → Review | Same as above, `source = "pipeline_trigger"` |

Multiple TimeEntry rows per task per day — handles interruptions correctly.

## Double-click Card → Task Panel

Slides in from right. Contains:
- Full task details (shot, project, assignee, due date)
- Estimated vs actual hours
- Status history (every transition with timestamp and who)
- Comments thread
- Pipeline events (render complete, publish events)
- Linked asset versions
- SyncSketch review link (if in review)
- Feedback from supervisor/producer

## Daily Hour Planning

At start of day (or on card open):
- Artist sets "planning to spend X hours today" per card
- Stored as `DailyAllocation.planned_hours`
- Compared against actual TimeEntry sum at end of day
- Feeds actuals vs plan reporting for producers

## Related
- [[Data Models]]
- [[Project View]]

---

# Notifications

tags: #feature #cross-cutting

---

## Delivery Mechanism

**In-app (QSystemTrayIcon)** — works when app minimised, OS native style.
**WebSocket** — server pushes events instantly, no polling. PySide6 uses `QWebSocket`.
**Celery hooks** — Slack message on high-priority events (blocked shots, deliveries).

## Event Types

| Event | Who notified | Channel |
|---|---|---|
| Task moved to Review | Supervisor | Tray + WebSocket |
| Task marked Blocked | Producer + Supervisor | Tray + Slack |
| Render complete | Artist | Tray |
| Feedback added to task | Assigned artist | Tray + WebSocket |
| Bid approved | Producer | Tray |
| Artist over allocated | Producer | Tray |
| Task overdue | Artist + Producer | Tray + Slack |

## Live UI Updates

WebSocket connection on app startup. On event received:
- Gantt bar updates colour/position without manual refresh
- Kanban card moves column if pipeline triggered
- Notification badge on relevant view tab

## Related
- [[Architecture Overview]]
- [[Artist Kanban View]]
