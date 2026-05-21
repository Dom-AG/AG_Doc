# Producer Calendar View

tags: #feature #priority-2 #producer #scheduler

> **Priority 2** — Xytech-style resource scheduler. People as rows, time as columns.

---

## Purpose

Give producers a single view of all people, what projects they're on, when, and whether capacity exists. Primary tool for scheduling and spotting conflicts.

---

## Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  [Week ◀ ▶]  [Month ◀ ▶]  [Today]    Filter: Dept ▾  Person ▾  │
├──────────────┬──────────────────────────────────────────────────┤
│ Person       │ Mon 3  Tue 4  Wed 5  Thu 6  Fri 7  ...          │
├──────────────┼──────────────────────────────────────────────────┤
│ Alice (Comp) │ ██████ PROJECT_A ██████ │ ███ PROJECT_B ███     │
│ Bob (FX TD)  │                         │ ████████ PROJECT_A ██ │
│ Carol (Light)│ ███ PROJECT_B ███        │                       │
├──────────────┼──────────────────────────────────────────────────┤
│ Unassigned   │  2 FX TD days available this week                │
└──────────────┴──────────────────────────────────────────────────┘
```

- Rows = People (grouped by department, collapsible)
- Columns = Days (zoom: day / week / month)
- Bars = Allocations (coloured per project)
- Unassigned row = remaining bid days not yet scheduled

---

## Interactions

**Drag bar** — move allocation to different dates
**Drag bar edge** — resize (extend or shorten allocation)
**Click bar** — show allocation detail popover (project, skill, days, bid line link)
**Double-click bar** — open full project panel
**Drag from Unassigned row** → person row — create new allocation
**Right-click bar** — context menu: Edit / Split / Delete

**Conflict detection:**
- Bars turn red outline if person double-booked on same dates
- Tooltip shows conflict detail
- Does not prevent saving — warns only

---

## Zoom Levels

| Level | Column width | Bar label |
|---|---|---|
| Day | Full day visible | Project code + task |
| Week (default) | 5-day columns | Project code |
| Month | Compressed columns | Project code abbreviated |
| Quarter | Very compressed | Colour only |

Zoom toggle in toolbar. Remembers last used per user.

---

## Capacity Indicators

Below each person row (or as row colour):
- **Green** — under 80% allocated this period
- **Amber** — 80–100% allocated
- **Red** — over-allocated (>100%)

Calculation: `sum(allocation.days) / available_working_days_in_period`

Available days excludes weekends and studio holidays (holiday calendar in settings).

---

## Filters + Grouping

- Filter by Department
- Filter by Project
- Filter by Person
- Group by: Department (default) / Project / Skill
- Show/hide: weekends, public holidays, unallocated rows

---

## Relationship to Bid Form

Each Allocation can optionally link to a BidLine:
- Shows whether bid days are covered by scheduled people
- Unlinked allocations = work happening outside bid scope (flag for producer)
- BidLine shows: `X days bid / Y days scheduled / Z days actual`

This is the core variance visibility Xytech provides.

---

## Open Questions

- [ ] **Holiday calendar** — studio-wide only, or per-person holidays too?
- [ ] **Part-day allocations** — can a person be on two projects same day (e.g. 50/50)?
- [ ] **Approval workflow** — does an allocation need supervisor sign-off before it's confirmed?
- [ ] **View permissions** — can artists see the full scheduler or only their own row?

---

## Related

- [[Data Models]]
- [[Bid Form Spec]]
- [[Project View]]
- [[People and Skills]]
