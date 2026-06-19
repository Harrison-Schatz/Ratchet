---
name: sizing-the-task
description: Use BEFORE starting any change request — feature, bugfix, refactor, config edit, doc change — no matter how small it looks, and BEFORE asking the user any clarifying question. Also use mid-task the moment an escalation trigger fires (more files than expected, a surprise, a risk surface appears, an acceptance check you can't write). Sizing decides how much process the task pays for.
---

# Sizing the Task

Assign the task a tier with objective criteria, so process weight is rule-governed instead of mood-governed. A typo fix and a new subsystem must not pay the same tax.

**Prevents:** failure mode #9 (mis-sized process — ceremony tax on small tasks, recklessness on big ones) and #2 (scope creep — the out-of-scope list starts here).

## Step 1 — Check the risk surfaces first

If the change touches ANY of these, the tier is **minimum 2**, regardless of size:

- auth / authz / session handling
- payments or anything money-adjacent
- data migration, deletion, or schema change
- secrets, keys, crypto
- concurrency primitives (locks, queues, shared state)
- public API contracts others depend on

## Step 2 — Walk the tier criteria top-down; first failure bumps you up

**Tier 0 — Patch.** ALL must hold:
- ≤ 2 files touched (your honest estimate)
- no interface, dependency, or behavior change beyond the literal request
- you can state the acceptance check in one sentence *without asking anyone* ("the README no longer says 'teh'")
- one `git revert` undoes it cleanly
- no risk surface

**Tier 1 — Task.** ALL must hold:
- one subsystem, ≤ ~5 files
- intent unambiguous: you can write the acceptance checks yourself and be confident the user would not be surprised by your interpretation
- no architectural decision, no new dependency
- no risk surface

**Tier 2 — Feature.** Any of: multiple subsystems, new components, new dependencies, genuine ambiguity (you'd be guessing at intent), or any risk surface.

**Tier 3 — Project.** The work cannot fit one session, or decomposes into multiple Tier 2 features.

When honestly unsure between two tiers, take the higher one — escalating later costs more than starting one tier up.

## Step 3 — Record the sizing

- **Tier 0:** no record needed before the work; the worklog line at the end covers it.
- **Tier 1:** append a **Definition of Done** to the task's `.ratchet/worklog/<task-id>.md` (create `.ratchet/` + `worklog/` if missing — see `keeping-state`):
  ```
  ## [2026-06-09 14:30] 2026-06-09-rate-limit-fix — sizing
  Tier 1. Goal: <one sentence>.
  Done when: <1-3 observable checks, e.g. "requests over 100/min return 429; test proves it">
  Out of scope: <what you are explicitly NOT doing, e.g. "no per-user config, no dashboard">
  ```
- **Tier 2/3:** same entry format, then update `.ratchet/STATE.md` with the new task id and phase `briefing`.

## Step 4 — Hand off

- **Tier 0:** implement now. Then `verifying-done`.
- **Tier 1:** implement per `testing-by-default`. Then `verifying-done`.
- **Tier 2/3:** invoke `writing-the-brief`.

## Re-sizing mid-task (the escape hatch)

Re-run this skill IMMEDIATELY when any trigger fires:

| Trigger | Typical move |
|---|---|
| Actual files touched exceeds the tier's bound | up one tier |
| You discover a risk surface mid-change | to Tier 2 minimum |
| You cannot write an acceptance check without guessing intent | to Tier 2 (you need a brief) |
| Second surprise on the same task (see `replanning-on-surprise`) | up one tier |
| The "feature" turned out to be a rename | down, with a stated reason |

Escalation procedure: stop work, append a worklog entry (`— escalation`, old tier → new tier, the trigger), keep all work that survives, then enter the new tier's path where it makes sense (usually `writing-the-brief` — the brief can be written around work already done). De-escalation requires the same entry with the reason.

## Stop conditions

- You cannot determine the tier because you don't understand the request at all → ask ONE clarifying question, then re-run sizing.
- The user explicitly dictates a process level ("just patch it, skip the questions") → honor it, record their instruction in the worklog entry, and still run `verifying-done` (evidence is never waived).

## Rationalization check

| Thought | Reality |
|---|---|
| "Obviously Tier 0, why write anything" | Tier 0 writes nothing up front — that IS the rule. But decide it, don't vibe it. |
| "Sizing is overhead" | It's five checkboxes. The overhead you fear is what sizing *prevents*. |
| "It's probably fine without the risk check" | The risk list is six bullets. Auth bugs ship exactly this way. |
