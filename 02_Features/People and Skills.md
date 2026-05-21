# People and Skills

tags: #feature #data #finance

> Foundation for bid forms, scheduling, and actuals. Must be set up before any financial tracking works.

---

## People

Fields:
- Name, email
- Department (FX / Comp / Lighting / Pipeline / Production)
- Active flag (deactivate rather than delete — preserves history)
- Default currency (GBP / USD)

## Skills

Studio-defined list — not free text. Examples:
- FX TD
- Compositor
- Lighter
- Pipeline TD
- Animator
- Roto / Paint
- Producer
- Supervisor

Department tag on each skill for grouping in bid form.

## Person ↔ Skill Rates

Each person has individual rates per skill they hold:

| Person | Skill | Day Rate | Currency | From | To |
|---|---|---|---|---|---|
| Alice | Compositor | £550 | GBP | 2024-01-01 | null |
| Alice | Supervisor | £700 | GBP | 2024-01-01 | null |
| Bob | FX TD | £650 | GBP | 2024-01-01 | null |
| Bob | FX TD | £600 | GBP | 2023-01-01 | 2023-12-31 |

Rate history preserved — old rates never deleted.
`effective_to = null` = currently active rate.

## UI — People Management Screen

- List view: all people, filterable by department, active status
- Click person → side panel with their skills and rates
- Add skill → pick from skill list, enter rate, currency, effective date
- Edit rate → creates new row with `effective_from = today`, closes old row
- Deactivate person → sets `active = false`, preserves all history

## Open Questions

- [ ] **Contractor vs staff** — same rate model or separate fields?
- [ ] **Overtime rate** — fixed amount or multiplier? Required on bid form?
- [ ] **Who can edit rates?** — Producer only, or can supervisors see rates?
- [ ] **Rate visibility** — can artists see their own rate in the app?

## Related
- [[Bid Form Spec]]
- [[Producer Calendar View]]
- [[Data Models]]
