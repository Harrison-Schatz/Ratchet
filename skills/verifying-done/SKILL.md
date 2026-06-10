---
name: verifying-done
description: Use BEFORE saying "done", "fixed", "complete", "passing", "works now", "ready to merge", or ANY synonym or implication of success — before committing the final state, opening a PR, marking a task complete, or telling the user good news. Also use when the user asks "is it done?" or "does it work?". The claim is blocked until the evidence exists.
---

# Verifying Done

"Done" is a claim about the world, and claims require evidence gathered *after* the work was finished. This gate is structural: a completion claim is legitimate only if a fresh evidence entry exists in the worklog. No entry, no claim — there is nothing to argue about.

**Prevents:** failure mode #3 (completion theater), #4 (untested happy path — the tier definitions force failure-path proof), #2 (scope creep — the diff check catches it at the exit).

## The gate procedure

1. **Identify the proving commands.** What, executed right now, would demonstrate each part of the claim? (Tier definitions below set the minimum.)
2. **Run them fresh.** Output from before your last edit is void. "It passed earlier" is history, not evidence.
3. **Read the output.** Exit code AND content. Count failures. A pass with new warnings is a finding, not a pass.
4. **Diff against declared scope.** `git diff <base>...HEAD --stat`. Every touched file traceable to the task's goal/plan? Anything matching the out-of-scope list? Out-of-scope edits → either revert them or escalate honestly to the user. Undeclared-but-defensible edits (drive-by typo fix) → declare them in the report.
5. **Write the evidence entry** in `.ratchet/WORKLOG.md`:
   ```
   ## [time] <task-id> — evidence (gate, T<n>)
   ran: npm test → 142 passed, 0 failed, exit 0
   ran: npm run lint → clean
   scope: 6 files, all within plan; plus README typo (declared)
   checks: brief items 1-5 demonstrated (1: curl output below; ...)
   ```
6. **Only now** state the result — WITH the evidence, including any caveats. If the evidence contradicts the claim, the truthful report ("2 tests still failing: X, Y") *is* the deliverable.

## What "done" means per tier

| Tier | The claim requires, minimum |
|---|---|
| **T0** | The single proving command/observation, run fresh, recorded in the one-line worklog `done` entry. (For a doc/typo change: the rendered/grep'd result.) If the repo has no `.ratchet/` directory, do NOT create one for a Tier 0 task — put the evidence line in the commit message and your report to the user instead. |
| **T1** | T0 + project test suite (no NEW failures vs. recorded baseline) + at least one test covering the change's failure path + diff-vs-DoD scope check |
| **T2** | T1 + full suite & lint/build + EVERY acceptance check in the brief individually demonstrated + `reviewing-the-diff` completed with Critical/Important findings resolved |
| **T3** | T2 per milestone; project-level: all milestone gates passed + the brief's milestone list reconciled |

A claim above your evidence is capped: tests passing but acceptance check 4 undemonstrated = "5 of 6 verified; #4 pending because X" — not "done."

## Hard rules

- **Subagent and tool reports are claims, not evidence.** An agent saying "all tests pass" obligates YOU to run the tests. (See `delegating-to-agents`.)
- **Verification cannot be waived** — not by the user's "just ship it" (then the *user's instruction* is recorded and evidence still gathered for what was actually checked), not by tier, not by deadline. Tier scales how MUCH evidence; never whether.
- **Satisfaction language counts as a claim.** "Great, that should do it!" asserts success. Hold it until after the gate.

## On failure at the gate

- Proving command fails → not done. Back to the loop; anything not understood → `debugging-to-root-cause`. Do not weaken the test, the check, or the claim's wording to pass the gate.
- An acceptance check turns out unverifiable as written → that's a brief defect: `replanning-on-surprise`, amend the check with the user's confirmation, then re-gate.

## Handoff

Gate passed → `landing-the-change`. Gate passed for a mid-plan step → back to `executing-with-checkpoints` (step evidence uses the same rules, lighter weight).

## Rationalization check

| Thought | Reality |
|---|---|
| "It should work now" | "Should" is a prediction. The gate trades predictions for output. |
| "I just ran it two edits ago" | Two edits ago is a different program. Fresh means after the last change. |
| "The subagent verified it" | The subagent *claimed* it. Run the command. |
| "It's a one-line change, what could break" | One-liners ship most outages. T0 evidence is one command — run it. |
| "User is in a hurry" | The fast honest answer is "verified: X works; Y unverified." Hurry changes the report, not the evidence. |
| "Linter passed so it builds" | Different tools prove different claims. Match the command to the claim. |
