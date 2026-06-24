---
name: keeping-state
description: Use whenever something worth surviving the session is learned or decided — a sizing, a decision, a surprise, evidence, a blocker — and at every phase boundary (sized → briefed → planned → executing step N → verifying → done). Also use when creating the .ratchet/ directory for the first time, or when unsure what belongs in STATE.md vs the worklog vs LESSONS.md.
---

# Keeping State

The conversation is volatile memory; `.ratchet/` is disk. Anything a fresh session would need gets written down *at the moment it's known* — not at the end, because the end is what a dying session never reaches.

**Prevents:** failure mode #5 (context loss between sessions). Everything `resuming-work` does depends on what this skill wrote.

## The layout

```
.ratchet/
├── STATE.md        # ROSTER — index of all active tasks (one row each). Overwritten.
├── state/          # <task-id>.md — one per active task: that task's full snapshot
├── worklog/        # <task-id>.md — one append-only journal per task
├── LESSONS.md      # project rules earned from retrospectives (see retrospecting)
├── briefs/         # <task-id>-brief.md
└── plans/          # <task-id>-plan.md
```

Task id: `YYYY-MM-DD-<slug>` (date the task started). Add `.ratchet/` to version control — state that isn't pushed dies with the laptop. (If the user objects, gitignore it and say the durability guarantee is now local-only.)

## What goes where — the only rule you need

- **Will a fresh session need it to act correctly *right now*?** → the task's `state/<task-id>.md` (and its roster row in STATE.md)
- **Might anyone need to know it happened?** → the task's `worklog/<task-id>.md`
- **Should every future session change behavior because of it?** → LESSONS.md (via `retrospecting` — don't write it directly mid-task)

## STATE.md — the roster (overwrite when the set of active tasks changes)

STATE.md is an INDEX of active tasks, not one task's snapshot. One row per active task;
the row's details live in `state/<task-id>.md`. Multiple active tasks are normal — two
agents in parallel, or one person juggling two threads.

```markdown
# Ratchet State — Roster
updated: 2026-06-09 15:42

## Active tasks
| task-id | owner | tier | phase | step | branch | state file | worklog |
|---|---|---|---|---|---|---|---|
| 2026-06-09-oauth-login | alice | 2 | executing | 3/7 | 2026-06-09-oauth-login | .ratchet/state/2026-06-09-oauth-login.md | .ratchet/worklog/2026-06-09-oauth-login.md |

## Open backlog (unowned)
- <items not tied to any active task>
```

## state/<task-id>.md — the per-task snapshot (overwrite at every phase boundary and executed step)

One file per active task, written ONLY by that task's owner — the old single-STATE body,
now per task:

```markdown
# Task state: 2026-06-09-oauth-login
updated: 2026-06-09 15:42
owner: alice
tier: 2
phase: executing          # sizing|briefing|planning|executing|verifying|landing|retro
step: 3 of 7
branch: 2026-06-09-oauth-login
owned paths: src/auth/**, tests/auth/**

## Next action
Implement step 4 (token refresh): see plan. Step 3 evidence is in the worklog.

## Blocked on
(nothing | the specific question + who can answer it)

## Pointers
brief:   .ratchet/briefs/2026-06-09-oauth-login-brief.md
plan:    .ratchet/plans/2026-06-09-oauth-login-plan.md
worklog: .ratchet/worklog/2026-06-09-oauth-login.md
```

"Next action" is the most important line in the system: ONE imperative sentence a stranger could execute. "Continue working" is a violation; "Run step 4 of the plan; note the refresh endpoint returns 204 not 200 (worklog 15:30)" is the standard. A task with no work left does not sit idle-in-place — it LANDS: its row leaves the roster and its state file is archived/removed (see `landing-the-change`). An empty roster means nothing is active.

## Ownership — stay in your lane

Each active task's state file lists its **owned paths** (and branch). Binding on every agent:

- Claim a task — a roster row + a `state/<task-id>.md` — before editing anything.
- Edit ONLY the paths/branch your task owns. Never edit, commit, or reset paths or a
  branch owned by another active task; find drift there → report it, don't fix it.
- Two active tasks must not own overlapping write-sets. If work needs the same files,
  it's one task (sequential), not two concurrent ones.

Agents usually share one working copy and branch space, so these rules are what keep
parallel work from clobbering — any project's LESSONS.md will reinforce them with its
own past collisions.

## Backward compatibility

An older `.ratchet/` may hold a single-snapshot `STATE.md` (one `## Current task`, no
`state/` dir). Treat it as a one-row roster: resume or land that task normally; on the
next state write, split it into `STATE.md` (roster) + `state/<task-id>.md`. No flag day.

Likewise, an older `.ratchet/` may hold a single append-only `WORKLOG.md` and no `worklog/`
dir. Treat it as legacy history: read it when resuming, leave it in place, and put NEW work in
`worklog/<task-id>.md` going forward. No flag day.

## worklog/<task-id>.md — append-only per-task entries

Each active task appends to its OWN `.ratchet/worklog/<task-id>.md` (born when the task is sized, or on
first write). The dated, task-id'd header on every entry keeps it self-identifying. A landed task's
worklog file stays in place as its permanent record — unlike the ephemeral state snapshot, the journal
is never deleted; the roster simply stops listing the task as active.

```markdown
## [2026-06-09 15:30] 2026-06-09-oauth-login — surprise
Google's refresh endpoint returns 204 with empty body, not the 200+JSON the plan
assumed. Plan step 4 amended (see plan change log).
```

Entry types: `sizing`, `decision`, `surprise`, `evidence`, `escalation`, `blocked`, `done`, `retro`. Keep entries to 1–4 lines; link to files rather than pasting them. **Evidence entries must contain the actual command and a verbatim result summary** — they are what `verifying-done` audits.

## When to write (checklist)

("worklog" below = the task's `.ratchet/worklog/<task-id>.md`.)

| Moment | Write |
|---|---|
| Task sized | worklog `sizing` (+ DoD for Tier 1) ; roster row + `state/<id>.md` if Tier 2+ |
| Brief approved / plan written | task state file: phase change |
| Plan step proven | worklog `evidence`; task state file: step tick + next action |
| Non-obvious choice made | worklog `decision` (one line: what + why) |
| Reality contradicted an assumption | worklog `surprise` (then `replanning-on-surprise`) |
| Waiting on the user | task state file `blocked` + worklog `blocked` |
| Gate passed, change landed | worklog `done`; row leaves roster; state file archived (the worklog file stays as the record) |

Tier 0 writes exactly one line to its `worklog/<task-id>.md` at the end (`done`, with its evidence) and touches nothing else. If no `.ratchet/` exists yet, Tier 0 does not create it — the evidence line goes in the commit message and the report to the user; the directory (and the task's `worklog/` file) is born at the first Tier 1+ task.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll remember it" | You are the one entity here guaranteed not to. |
| "I'll write it all up at the end" | Sessions die mid-task. That's the one scenario state exists for. |
| "This is a lot of writing" | Per event it's 1–4 lines, ~30 seconds. The alternative is a future session re-deriving it at full cost — or getting it wrong. |
