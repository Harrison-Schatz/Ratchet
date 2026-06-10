---
name: resuming-work
description: Use at the start of any session where .ratchet/STATE.md shows status active or blocked, when the user says "continue", "pick up where we left off", "where were we", or references work you have no memory of, or after a context compaction mid-task. Resume from disk — do not reconstruct from guesswork or restart work that already exists.
---

# Resuming Work

A fresh session knows nothing. The previous session wrote everything needed to disk. Resume = read, verify reality still matches, reconcile, continue. Never trust STATE.md blindly — the session may have died between doing work and recording it.

**Prevents:** failure mode #5 (context loss — specifically the failure where a fresh session restarts, duplicates, or contradicts past work).

## Step 1 — Read the record

In order:
1. `.ratchet/STATE.md` — task, tier, phase, step, next action, blockers.
2. `.ratchet/LESSONS.md` — rules that bind you this session.
3. The current task's brief and plan (pointers in STATE.md), including the plan's **change log**.
4. The task's worklog entries (grep WORKLOG.md for the task id) — decisions and surprises are context the plan alone won't give you.

## Step 2 — Verify reality matches the record

STATE.md is a *claim*. Check it:

```
git status                  # uncommitted changes? on the recorded branch?
git log --oneline -10       # do commits match the plan's checkpointed steps?
<project test command>      # does the suite state match the last evidence entry?
```

| Finding | Meaning | Action |
|---|---|---|
| Repo matches STATE.md exactly | clean death at a checkpoint | continue from "Next action" |
| Work exists beyond the last recorded step (uncommitted changes, extra commits) | died mid-step, after working but before recording | Read the diff. If the partial step is verifiable, finish its checkpoint (commit + evidence + STATE tick). If half-broken, prefer reverting to the last checkpoint over guessing intent — the plan can re-derive the step; a misunderstood half-edit can't be trusted. |
| STATE.md claims more than the repo shows | recorded optimistically, or wrong branch | trust the repo; correct STATE.md; log a `decision` entry noting the correction |
| Tests now fail that the last evidence entry showed passing | environment drift or unrecorded change | `debugging-to-root-cause` before any new work |

## Step 3 — Reconcile and announce

1. Correct STATE.md to match verified reality (this is the one case where you rewrite it outside a phase boundary).
2. Append a worklog entry: `— decision: resumed; repo state <matched | reconciled: how>`.
3. Tell the user in 2–4 sentences where things stand: task, step N of M, anything reconciled, what you'll do next. Do not re-litigate decisions already recorded in the brief/worklog — they were made; the record is the memory.

## Step 4 — Continue

- `status: blocked` → check whether the blocking answer has arrived (in the user's message, or overtaken by events). If not, ask it again — once — and stop.
- phase `executing` → re-enter `executing-with-checkpoints` at the recorded step.
- phase `briefing`/`planning` → re-enter that skill where its artifact left off.
- phase `verifying`/`landing` → `verifying-done` (gates re-run from scratch; stale evidence is no evidence).
- `status: idle` but the user references unfinished work → the record and the user disagree; show them the last `done` entry and ask what they're seeing. Their answer is probably a new task — `sizing-the-task`.

## Stop conditions

- No `.ratchet/` but the user says "continue": say plainly that no state exists on disk, summarize what git history shows, and ask what they want carried forward. Do not fabricate continuity.
- STATE.md is internally contradictory or stale beyond reconciliation (>1 task tangled): present what you verified from the repo, propose a corrected STATE.md, get a nod before proceeding.

## Rationalization check

| Thought | Reality |
|---|---|
| "I can infer where things stand from the code" | The code shows *what*; the worklog shows *why* and *what was rejected*. Skipping it re-makes settled decisions, wrongly. |
| "STATE.md says step 5, so step 5" | STATE.md is a claim by a dead session. Verify against git and tests first — two minutes. |
| "I'd rather start fresh, it's cleaner" | Restarting discards verified, committed progress. The ratchet only works if clicks hold. |
