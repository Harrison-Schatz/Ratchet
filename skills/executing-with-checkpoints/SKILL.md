---
name: executing-with-checkpoints
description: Use when a Tier 2/3 plan exists and it's time to build — "execute the plan", "go ahead", "start implementing". Runs the plan step by step with durable checkpoints so a dead session loses at most one step. Do not freestyle a Tier 2+ implementation outside this loop.
---

# Executing with Checkpoints

Run the plan one step at a time. Every step ends in a provable, committed, recorded state — that's the ratchet click. Between clicks you can think freely; at clicks, the state is on disk.

**Prevents:** failure mode #5 (context loss — at most one step of work is ever unrecorded) and #7 (drift — surprises route to replanning instead of silent improvisation).

## Setup (once)

1. Confirm you are NOT on the default branch; branch if needed (`git switch -c <task-id>` or the harness's native isolation). If the project defines its own isolation runbook or skill (dev containers, per-agent worktree stacks), use that instead of inventing a setup — LESSONS.md or project docs will name it.
2. Run the project's test suite to record the baseline in the worklog:
   - **Green** → proceed.
   - **Red** → STOP. Record which tests already fail (these are not yours to fix silently, and not yours to worsen). Ask the user whether to proceed-around or fix-first, unless LESSONS.md already answers it. Your gate later is "no NEW failures."
   - **No suite exists** → record `baseline: no test infrastructure` and confirm the plan's first step bootstraps a minimal runner (per `planning-the-work` Step 1); the gate criterion becomes "the suite that exists by the end passes."
3. Update `.ratchet/STATE.md`: phase `executing`, step 0 of N.

## The loop — for each plan step

1. **Read the step.** If anything is ambiguous or contradicts what you see in the repo → do NOT improvise; invoke `replanning-on-surprise`.
2. **Implement** under the step's declared test discipline (`testing-by-default`; in untested territory `characterizing-legacy-code` first).
3. **Prove it.** Run the step's "Prove it" command. Read the output. Failing → fix or, if the *approach* is wrong, `replanning-on-surprise`. Anything broken you don't understand → `debugging-to-root-cause`.
4. **Checkpoint** (the click — all four, in order):
   1. Commit with the step's message.
   2. Append worklog evidence line: `## [time] <task-id> — evidence` + the command run + result summary ("14 passed, 0 failed").
   3. Tick the step in STATE.md (`step 3 of 7`), update "Next action" to the next step's name.
   4. One line for anything learned that a fresh session would need (decisions, gotchas) — `keeping-state` has the formats.
5. **Next step.** Do not pause to ask "should I continue?" — the plan was the authorization. Stop only for the stop conditions below.

## Stop conditions (the ONLY reasons to halt the loop)

- An escalation trigger fires (`sizing-the-task`: blown estimates, new risk surface, second surprise).
- A step is blocked on information only the user has → ask the specific question, record it under STATE.md "Blocked on", stop.
- Evidence contradicts the brief's assumptions → `replanning-on-surprise`.
- All steps done → proceed to **Finish** below.

"I'm not sure this is what they meant" on a *brief-level* question is a stop. On a *code-level* choice within the brief's intent, decide, record the decision in the worklog, and continue.

## Delegation option

Steps marked parallel-safe in the plan may be dispatched via `delegating-to-agents` (fresh-context subagent per step, never trusting reports — that skill owns the rules). The checkpoint discipline is identical: no step is "done" until YOU have verified its proof command and written the evidence line yourself.

## Finish

When the last step's checkpoint is written:
1. Update STATE.md: phase `verifying`.
2. Invoke `verifying-done` (T2 gate: full suite, acceptance checks, scope diff).
3. Then `reviewing-the-diff`, then `landing-the-change`. Do not announce completion before those gates — the words "done", "complete", "working" are owned by `verifying-done`.

## Rationalization check

| Thought | Reality |
|---|---|
| "Committing every step is noisy" | Squash at landing if the project prefers. Durable beats tidy mid-flight. |
| "I'll update STATE.md at the end" | The end is exactly what a dying session never reaches. Update at the click. |
| "The plan says X but Y is clearly better" | Maybe! That's a change-log entry via replanning-on-surprise — 2 minutes — not a silent fork. |
