---
name: keeping-state
description: Use whenever something worth surviving the session is learned or decided — a sizing, a decision, a surprise, evidence, a blocker — and at every phase boundary (sized → briefed → planned → executing step N → verifying → done). Also use when creating the .ratchet/ directory for the first time, or when unsure what belongs in STATE.md vs WORKLOG.md vs LESSONS.md.
---

# Keeping State

The conversation is volatile memory; `.ratchet/` is disk. Anything a fresh session would need gets written down *at the moment it's known* — not at the end, because the end is what a dying session never reaches.

**Prevents:** failure mode #5 (context loss between sessions). Everything `resuming-work` does depends on what this skill wrote.

## The layout

```
.ratchet/
├── STATE.md        # current snapshot — OVERWRITTEN, always reflects "now"
├── WORKLOG.md      # append-only journal — NEVER edited, only appended
├── LESSONS.md      # project rules earned from retrospectives (see retrospecting)
├── briefs/         # <task-id>-brief.md
└── plans/          # <task-id>-plan.md
```

Task id: `YYYY-MM-DD-<slug>` (date the task started). Add `.ratchet/` to version control — state that isn't pushed dies with the laptop. (If the user objects, gitignore it and say the durability guarantee is now local-only.)

## What goes where — the only rule you need

- **Will a fresh session need it to act correctly *right now*?** → STATE.md
- **Might anyone need to know it happened?** → WORKLOG.md
- **Should every future session change behavior because of it?** → LESSONS.md (via `retrospecting` — don't write it directly mid-task)

## STATE.md — overwrite on every phase boundary and every executed step

```markdown
# Ratchet State
updated: 2026-06-09 15:42
status: active            # active | blocked | idle

## Current task
id: 2026-06-09-oauth-login
tier: 2
phase: executing          # sizing|briefing|planning|executing|verifying|landing|retro
step: 3 of 7
branch: 2026-06-09-oauth-login

## Next action
Implement step 4 (token refresh): see plan. Step 3 evidence is in the worklog.

## Blocked on
(nothing | the specific question + who can answer it)

## Pointers
brief: .ratchet/briefs/2026-06-09-oauth-login-brief.md
plan:  .ratchet/plans/2026-06-09-oauth-login-plan.md
```

"Next action" is the most important line in the system: ONE imperative sentence a stranger could execute. "Continue working" is a violation; "Run step 4 of the plan; note the refresh endpoint returns 204 not 200 (worklog 15:30)" is the standard. When no work is in flight: `status: idle`, task block empty.

## WORKLOG.md — append-only entries

```markdown
## [2026-06-09 15:30] 2026-06-09-oauth-login — surprise
Google's refresh endpoint returns 204 with empty body, not the 200+JSON the plan
assumed. Plan step 4 amended (see plan change log).
```

Entry types: `sizing`, `decision`, `surprise`, `evidence`, `escalation`, `blocked`, `done`, `retro`. Keep entries to 1–4 lines; link to files rather than pasting them. **Evidence entries must contain the actual command and a verbatim result summary** — they are what `verifying-done` audits.

## When to write (checklist)

| Moment | Write |
|---|---|
| Task sized | worklog `sizing` (+ DoD for Tier 1) ; STATE.md if Tier 2+ |
| Brief approved / plan written | STATE.md phase change |
| Plan step proven | worklog `evidence`; STATE.md step tick + next action |
| Non-obvious choice made | worklog `decision` (one line: what + why) |
| Reality contradicted an assumption | worklog `surprise` (then `replanning-on-surprise`) |
| Waiting on the user | STATE.md `blocked` + worklog `blocked` |
| Gate passed, change landed | worklog `done`; STATE.md → idle |

Tier 0 writes exactly one worklog line at the end (`done`, with its evidence) and touches nothing else. If no `.ratchet/` exists yet, Tier 0 does not create it — the evidence line goes in the commit message and the report to the user; the directory is born at the first Tier 1+ task.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll remember it" | You are the one entity here guaranteed not to. |
| "I'll write it all up at the end" | Sessions die mid-task. That's the one scenario state exists for. |
| "This is a lot of writing" | Per event it's 1–4 lines, ~30 seconds. The alternative is a future session re-deriving it at full cost — or getting it wrong. |
