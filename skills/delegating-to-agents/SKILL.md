---
name: delegating-to-agents
description: Use when work can be split across subagents — parallel-safe plan steps, multiple independent bug investigations, a broad codebase survey — or when the main session's context is filling with detail a delegate could absorb instead. Also use BEFORE accepting any subagent's "success" report. Covers when to delegate, how to brief a delegate, and the no-trust verification rule.
---

# Delegating to Agents

A subagent is a fresh context: it knows exactly what you tell it, nothing else. That's its weakness (zero project memory) and its power (zero accumulated confusion, and your context stays clean for coordination). Delegation pays when the brief is cheap relative to the work; it fails when delegates collide or when their reports are believed.

**Prevents:** failure mode #3 in its delegated form (subagent completion theater — the most common kind) and context exhaustion on long tasks (#5's cousin).

## When to delegate — and when not

**Delegate:** plan steps marked parallel-safe (disjoint files, no ordering); 2+ independent investigations (different failing test files, different subsystems); bulk mechanical work matching one description (apply pattern to N sites); wide read-only research whose detail you don't need in your context, only the conclusion.

**Don't delegate:** tightly coupled steps (delegate B needs decisions A is still making); work needing accumulated session judgment (brief-writing, replanning, anything user-facing); tasks whose specification would be longer than doing it; **never two agents writing the same files concurrently** — shared files mean sequential or one agent.

## Step 1 — Brief the delegate (it knows NOTHING)

Every dispatch contains, pasted inline (a delegate told to "read the plan" reads it without the conversation that gave it meaning):
1. **Goal** — one sentence.
2. **Full task text** — the plan step / DoD verbatim, plus only the context that makes it actionable (relevant file paths, the surprising API behavior from the worklog, the project's test command, applicable LESSONS.md rules).
3. **Constraints** — files it may touch; files it must NOT touch; discipline expected (`test-first per RED-GREEN`); "do NOT fix unrelated things you find — report them."
4. **Output contract** — exactly what to return: files changed, commands run **with verbatim output**, what was NOT done, surprises encountered.
5. **Escalation permission** — "if blocked or the task seems wrong, STOP and say so; a clear question beats a confident mess."

## Step 2 — Dispatch

- Parallel only when write-sets are disjoint AND no result feeds another. Otherwise sequential.
- Environment isolation is project-specific: if the project defines an isolation runbook or skill (per-agent worktrees, dev-container stacks with their own DB/ports), use it for parallel delegates. Note its limit honestly — it removes *runtime* collisions (files, test DB, ports), not *semantic* ones; the cross-delegate suite check in Step 3 still applies when the branches meet.
- Match capability to the task: mechanical/well-specified → cheaper model is fine; judgment/design/review → most capable.
- Keep the count honest: each delegate's work must be individually verified by you (Step 3) — dispatch only what you can audit.

## Step 3 — Verify like the gate, because it is the gate

**A delegate's report is a claim. Evidence is what YOU observe.** For each returned task:
1. Look at the actual diff (`git diff` / read the files). Changes exist? Within the allowed file set?
2. Run the proving command yourself (the plan step's "Prove it", or the suite). The delegate's pasted output is *its* evidence; the gate needs *yours* — agents misread their own output, run stale commands, or test the wrong thing, without lying once.
3. Cross-delegate check after parallel work: full suite once (delegates can each pass alone and conflict together), plus skim for overlapping edits.
4. Only then checkpoint per `executing-with-checkpoints` (commit, evidence line, STATE tick — the evidence line cites YOUR run).

Report says DONE but the diff is empty or the proof fails → the task is NOT done. Re-dispatch with the specific gap named, or do it yourself. Never paper over a failed delegation by "fixing it up" silently — the worklog records what happened.

## Handling delegate outcomes

| Outcome | Move |
|---|---|
| Done, verified | Checkpoint; next |
| Done with concerns | Read the concerns FIRST; correctness/scope concerns block the checkpoint until resolved |
| Blocked / needs context | Answer and re-dispatch; missing context is the dispatcher's bug, fix the brief not the delegate |
| Wrong because the STEP is wrong | Not a delegation failure — `replanning-on-surprise` |
| Failed twice on a well-briefed task | Stop re-rolling the dice: the task is under-specified or too big — split it, or do it inline where judgment lives |

## Rationalization check

| Thought | Reality |
|---|---|
| "The agent said all tests pass" | Then it will cost you ten seconds to see the same thing. Run it. The one time in ten it doesn't reproduce pays for the other nine. |
| "Re-verifying everything defeats the point of delegating" | The point was parallel HANDS, not delegated ACCOUNTABILITY. Verification is the part that was never delegable. |
| "Both agents passed their own tests" | Each in a world where the other doesn't exist. The merged world runs the suite once more. |
| "It'll be faster to spawn five agents for this" | Five briefs + five audits. Honest math first; delegation has fixed costs per delegate. |
