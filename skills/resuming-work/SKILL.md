---
name: resuming-work
description: Use at the start of any session where .ratchet/STATE.md shows status active or blocked, when the user says "continue", "pick up where we left off", "where were we", or references work you have no memory of, or after a context compaction mid-task. Resume from disk — do not reconstruct from guesswork or restart work that already exists.
---

# Resuming Work

A fresh session knows nothing. The previous session wrote everything needed to disk. Resume = read, verify reality still matches, reconcile, continue. Never trust STATE.md blindly — the session may have died between doing work and recording it.

**Prevents:** failure mode #5 (context loss — specifically the failure where a fresh session restarts, duplicates, or contradicts past work).

## Step 1 — Read the record

First identify WHICH task you're resuming: the user names it, you own a row in the
roster, or there's exactly one active row. Then, in order:
1. `.ratchet/STATE.md` (roster) → find your task's row, then read `.ratchet/state/<task-id>.md` — phase, step, next action, blockers, owned paths.
2. `.ratchet/LESSONS.md` — rules that bind you this session.
3. That task's brief and plan (pointers in its state file), including the plan's **change log**.
4. The task's worklog file `.ratchet/worklog/<task-id>.md` — decisions and surprises are context the plan alone won't give you.

## Step 2 — Verify reality matches the record

The state file is a *claim*. Check it:

```
git status                  # uncommitted changes? on the recorded branch?
git log --oneline -10       # do commits match the plan's checkpointed steps?
<project test command>      # does the suite state match the last evidence entry?
```

| Finding | Meaning | Action |
|---|---|---|
| Repo matches the state file exactly | clean death at a checkpoint | continue from "Next action" |
| Work exists beyond the last recorded step (uncommitted changes, extra commits) | died mid-step, after working but before recording | Read the diff. If the partial step is verifiable, finish its checkpoint (commit + evidence + state-file tick). If half-broken, prefer reverting to the last checkpoint over guessing intent — the plan can re-derive the step; a misunderstood half-edit can't be trusted. |
| The state file claims more than the repo shows | recorded optimistically, or wrong branch | trust the repo; correct the state file; log a `decision` entry noting the correction |
| Tests now fail that the last evidence entry showed passing | environment drift or unrecorded change | `debugging-to-root-cause` before any new work |

## Step 3 — Reconcile and announce

1. Correct your task's state file (and its roster row) to match verified reality (this is the one case where you rewrite outside a phase boundary). Touch only your task's files — never another active task's.
2. Append a worklog entry: `— decision: resumed; repo state <matched | reconciled: how>`.
3. Tell the user in 2–4 sentences where things stand: task, step N of M, anything reconciled, what you'll do next. Do not re-litigate decisions already recorded in the brief/worklog — they were made; the record is the memory.

## Step 4 — Continue

- `status: blocked` → check whether the blocking answer has arrived (in the user's message, or overtaken by events). If not, ask it again — once — and stop.
- phase `executing` → re-enter `executing-with-checkpoints` at the recorded step.
- phase `briefing`/`planning` → re-enter that skill where its artifact left off.
- phase `verifying`/`landing` → `verifying-done` (gates re-run from scratch; stale evidence is no evidence).
- the task is no longer on the roster (already landed) but the user references unfinished work → the record and the user disagree; show them the last `done` entry and ask what they're seeing. Their answer is probably a new task — `sizing-the-task`.

## Stop conditions

- No `.ratchet/` but the user says "continue": say plainly that no state exists on disk, summarize what git history shows, and ask what they want carried forward. Do not fabricate continuity.
- A single task's state file is internally contradictory or stale beyond reconciliation: present what you verified from the repo, propose a corrected state file, get a nod before proceeding. (Multiple active tasks in the roster is NOT this case — that's normal; resume only the one you own, leaving the others' files byte-unchanged.)

## Rationalization check

| Thought | Reality |
|---|---|
| "I can infer where things stand from the code" | The code shows *what*; the worklog shows *why* and *what was rejected*. Skipping it re-makes settled decisions, wrongly. |
| "The state file says step 5, so step 5" | It's a claim by a dead session. Verify against git and tests first — two minutes. |
| "I'd rather start fresh, it's cleaner" | Restarting discards verified, committed progress. The ratchet only works if clicks hold. |
