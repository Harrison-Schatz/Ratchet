---
name: planning-the-work
description: Use when a Tier 2/3 brief has been approved and before any implementation code is written. Also use when handed an external spec or requirements doc for multi-step work. Produces the step sequence that executing-with-checkpoints will run — do not start building a Tier 2+ task from the brief alone.
---

# Planning the Work

Decompose the brief into verifiable steps a zero-context executor could run. The plan specifies *intent, files, and proof* per step — it does not pre-write the implementation. Plans are hypotheses: they have a change-log because reality will edit them.

**Prevents:** failure mode #7 (plan drift — the change-log and checkpoint structure make amendment normal) and executor ambiguity (steps a stranger can't run are steps that get improvised).

## Step 1 — Map the territory

Before writing steps, list in scratch form: which files will be created/modified, what each is responsible for, what order dependencies force. In brownfield code, check now whether the touched code has tests — any step touching untested code must begin with `characterizing-legacy-code`, and that goes IN the plan. If the project has NO test infrastructure at all (no runner, no test script), the plan's Step 1 is bootstrapping a minimal runner (one installed dev-dependency, one passing smoke test, wired to the conventional command) — characterization and test-first both stand on it.

## Step 2 — Write the plan

Save to `.ratchet/plans/<task-id>-plan.md`:

```markdown
# Plan: <title>                    <!-- task-id matches the brief -->
Brief: .ratchet/briefs/<task-id>-brief.md

## Steps
### Step 1: <outcome, not activity — "rate limiter middleware exists and rejects >100/min">
- Files: `src/middleware/rateLimit.ts` (create), `src/app.ts` (wire in)
- Approach: <2-4 sentences. Sketch interfaces/signatures if a stranger would need them.
  Do NOT paste full implementations — that's the executor's job, test-first.>
- Test discipline: <test-first | characterize-first | spike-then-stabilize, per testing-by-default>
- Prove it: `npm test -- rateLimit` passes, including the over-limit failure case
- Checkpoint: commit "feat: rate limiter middleware"

### Step 2: ...

## Step ordering
<one line on why this order; which steps could run in parallel safely (for delegating-to-agents)>

## Change log
<empty at birth. Every amendment appends: date, what changed, why, which steps were
 invalidated/added. Maintained by replanning-on-surprise.>
```

**Granularity rule:** a step is the smallest unit that ends in a *provable, committable state* — typically 15–90 minutes of work, not 2 minutes. Wrong granularity signs: a step whose proof is "code compiles" (too small), a step touching three subsystems (too big).

**No placeholders.** "TBD", "handle errors appropriately", "similar to step 2" are plan failures. Every step names exact files and an executable proof. If you can't write the proof, you don't understand the step — return to the brief.

## Step 3 — Self-check the plan

1. **Coverage:** every acceptance check in the brief maps to ≥1 step; every step serves ≥1 check (or is enabling infrastructure — say so).
2. **Stranger test:** could an agent with no session memory run step 1 from the text alone? Each step?
3. **Consistency:** names/signatures referenced in later steps are defined in earlier ones.
4. **First-step reality check:** step 1 should be runnable *now* — if it depends on something unverified ("assuming the API supports X"), make verifying that assumption step 1.

Fix inline; no re-review ceremony.

## Step 4 — Hand off

Update `.ratchet/STATE.md`: phase `executing`, step `0 of N`, pointer to the plan. Then invoke `executing-with-checkpoints`. Plan approval by the user is NOT required (the brief was the intent gate) — but show the plan path and one-paragraph summary so they can object cheaply.

## Stop conditions

- A step requires an architectural decision the brief didn't settle → back to `writing-the-brief` (amend, reconfirm that section only).
- The plan exceeds ~10 steps → the task is Tier 3 in disguise; re-run `sizing-the-task` and decompose into milestones.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll just include the full code in each step, it's clearer" | You're writing the implementation blind, twice. The executor inherits your unverified guesses as gospel. Intent + files + proof. |
| "Plans slow me down, I know what to do" | Then the plan is short and took ten minutes. Its real job is surviving your context dying at step 4. |
| "I'll fix the ordering as I go" | Going out of order silently is drift. Reorder via the change log — it's one line. |
