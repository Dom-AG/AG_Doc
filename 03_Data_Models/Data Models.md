# Data Models

tags: #data #schema #decisions

> Core schema decisions. Changes here affect every view — treat as high risk.
> All schema changes via Alembic migration with written rationale. No direct prod edits.

---

## People + Skills + Rates

```
Person
  id, name, email, department, active

Skill
  id, name              # "Compositor", "FX TD", "Lighter", "Pipeline TD"
  department            # grouping for bid form display

PersonSkill             # one person can hold multiple skills
  person_id
  skill_id
  day_rate              # their rate for THIS skill specifically
  overtime_rate         # optional fixed amount or multiplier
  currency              # GBP or USD
  effective_from        # rate history — rates change over time
  effective_to          # null = currently active rate
```

**Key decisions:**
- Rates are **per person per skill** — not global skill rates
- Rate history preserved via `effective_from/to` — never delete old rates
- One person, multiple skills, different rate per skill

---

## Bid Form

```
BidForm
  id
  project_id
  version               # v1, v2, v3 — bids get revised
  status                # draft | sent | approved | rejected
  created_by (person_id)
  created_at
  notes                 # assumptions, exclusions
  currency              # primary currency for this bid

BidLine
  id
  bid_form_id
  skill_id              # role being bid (not a person yet)
  days_estimated
  rate_snapshot         # copied from PersonSkill.day_rate at bid time, FROZEN
  currency_snapshot     # frozen at bid time
  phase                 # "Prep" | "Production" | "Delivery"
  department
  notes
  # computed: days_estimated × rate_snapshot = line_total
```

**Key decisions:**
- Bid in **roles/skills**, not people — people assigned later in scheduler
- `rate_snapshot` frozen at bid time — historical bids never silently recalculate
- Version history required — clients revise scope, you need audit trail
- ⚠️ Confirm with Charlotte: **days or hours** as bid unit? Default assumption = days

---

## Projects + Shots

```
Project
  id, name, code        # code = "EPIC" used in env vars
  client_id
  status                # pitch | active | delivery | archived
  root_path             # /mnt/aslon/projects/CLIENT/PROJECT
  start_date, end_date
  base_packages (JSONB) # Rez packages
  dcc_envs (JSONB)      # per-DCC Rez environments

Shot
  id, project_id
  name, code
  status                # pending | in_progress | review | approved | omit
  cut_in, cut_out       # frame range
  complexity            # simple | standard | complex
  department

Task
  id, shot_id (or asset_id)
  skill_id              # what skill this task requires
  assigned_to (person_id)
  status                # pending | in_progress | review | approved | blocked
  due_date
  started_at            # set when card dragged to In Progress
  stopped_at            # set when card dragged to Review or pipeline triggers
  priority
  notes
```

---

## Scheduling + Actuals

```
Allocation              # scheduler assignment — person to project/dates
  id
  person_id
  skill_id              # which hat they're wearing on this project
  project_id
  bid_line_id           # optional link back to what bid line this covers
  start_date, end_date
  days_allocated
  notes

TimeEntry               # actual time booked — multiple per task per day
  id
  person_id
  task_id
  date
  started_at            # timestamp
  stopped_at            # timestamp (null if still running)
  hours                 # computed from start/stop or manual entry
  source                # "kanban_drag" | "pipeline_trigger" | "manual"
```

**Why TimeEntry not just start/stop on Task:**
Artists get pulled away mid-task — meetings, reviews, context switches.
Multiple TimeEntry rows per task per day captures real actuals accurately.

---

## Assets + Versions

```
Asset
  id, name, type, project_id, department
  status, metadata (JSONB)

AssetVersion
  id, asset_id
  version_number
  rez_context           # exact Rez resolve used to publish
  source_scene          # path to originating scene file
  publish_path          # path to published output
  published_by (person_id)
  published_at
  notes, thumbnail

VersionDependency
  upstream_version_id
  downstream_version_id
  # enables: "what does this cache affect?"

TaskAsset
  task_id, asset_id
  role                  # "input" | "output"
```

---

## The Bid → Schedule → Actuals Chain

```
BidLine (Skill + Days + Rate)       what you promised the client
    ↓
Allocation (Person + Skill + Dates) who you're putting on it
    ↓
TimeEntry (Person + Task + Hours)   what actually happened
```

Variance = gap between these three layers.
Core reporting question: **are we on track against bid, and who is over/under?**

---

## Cost Rollup

```
TimeEntry.hours × PersonSkill.day_rate / hours_per_day = cost per entry
    ↓ roll up to Task
    ↓ roll up to Shot
    ↓ roll up to Department
    ↓ roll up to Project
    ↓ compare to BidForm total
```

Computed nightly via Celery Beat — not live on every UI render.

---

## Related

- [[Bid Form Spec]]
- [[Producer Calendar View]]
- [[Artist Kanban View]]
- [[Architecture Overview]]
