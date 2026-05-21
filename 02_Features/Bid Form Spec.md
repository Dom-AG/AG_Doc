# Bid Form Spec

tags: #feature #priority-1 #producer #finance

> **Priority 1** — agreed with Charlotte. Foundation for all financial tracking.

---

## Purpose

Single source of truth for what was promised to the client, at what cost, broken down by role and phase. Feeds scheduler, actuals tracking, and invoicing.

---

## User Flow

```
New Project created
    ↓
Producer opens Bid Form for project
    ↓
Fills in header (client, currency, assumptions)
    ↓
Adds BidLines — one per role per phase
  → picks Skill from dropdown
  → enters days estimated
  → rate auto-fills from that person's current rate (or override)
  → line total auto-calculates
    ↓
Reviews totals by department and phase
    ↓
Marks as "Sent" — rate snapshot frozen
    ↓
Client approves → status = "Approved" → project activates
    ↓
Revisions → new version created, previous preserved
```

---

## UI Layout

### Header Section
- Project name (read-only, pulled from Project)
- Client name (read-only)
- Bid version (auto-incremented)
- Status badge (Draft / Sent / Approved / Rejected)
- Primary currency (GBP / USD)
- Created by, date
- Notes / Assumptions field (multi-line, important for scope clarity)

### Line Items Table

| Phase | Department | Skill | Days | Day Rate | Currency | Total | Notes |
|---|---|---|---|---|---|---|---|
| Prep | FX | FX TD | 5 | £650 | GBP | £3,250 | |
| Production | FX | FX TD | 15 | £650 | GBP | £9,750 | |
| Production | Comp | Compositor | 10 | £550 | GBP | £5,500 | |
| Delivery | Comp | Compositor | 3 | £550 | GBP | £1,650 | |

- Rows grouped by Phase, then Department
- Add row button per group
- Inline editing — click cell to edit
- Rate field: auto-fills from PersonSkill on Skill select, manually overridable
- Delete row with confirmation

### Totals Panel (right side or footer)
- Subtotal per department
- Subtotal per phase
- Grand total
- Multi-currency warning if mixed currencies on same bid

---

## Bid Versions

Every time a bid is revised:
- Current version marked as superseded
- New version created with all lines copied
- Version selector in header (v1, v2, v3...)
- Old versions read-only
- Diff view: highlight what changed between versions (stretch goal)

---

## Rate Handling

On Skill select in a BidLine:
1. Look up active `PersonSkill` rates for that skill
2. Show most common/median rate as default (no person assigned yet at bid stage)
3. Allow manual override — override stored on BidLine, not on PersonSkill

On status → "Sent":
- `rate_snapshot` written to every BidLine
- Rates frozen — PersonSkill changes do not affect this bid

---

## PDF Export

Producers need to send bids to clients. Required fields on export:
- Studio header/logo
- Client + project name
- Bid version + date
- Line items table (same as UI, minus internal notes)
- Totals
- Assumptions / notes section
- Signature / approval line

Use ReportLab or WeasyPrint via FastAPI endpoint — PySide6 triggers download.

---

## Open Questions

- [ ] **Days or hours as bid unit?** — confirm with Charlotte. Current assumption = days.
- [ ] **Overtime rate on bid lines?** — include or keep simple for v1?
- [ ] **Tax handling on PDF export?** — VAT line item needed for UK clients?
- [ ] **Multi-currency bids** — single currency per bid or mixed? Xero handles FX but bid form needs a stance.
- [ ] **Who can create/edit bids?** — Producer only, or Supervisors too?

---

## Related

- [[Data Models]]
- [[Producer Calendar View]]
- [[People and Skills]]
