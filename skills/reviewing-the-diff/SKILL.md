---
name: reviewing-the-diff
description: Use when Tier 2/3 implementation is complete (before the final verifying-done gate), when any change touches a risk surface (auth, payments, data migration, secrets, concurrency, public APIs), or when review feedback ARRIVES from a human or tool and you're deciding what to do with it. One skill, two motions: running a review, and receiving one.
---

# Reviewing the Diff

Review is a fresh set of eyes on the diff before it becomes permanent. Ratchet runs ONE review with two checklists — does it match intent, and is it well built — rather than two ceremonial passes. Risk, not ritual, decides depth.

**Prevents:** failure mode #2 (scope creep caught at the exit), #1's late echo (intent misses caught before merge), and — in the receiving motion — wrong "fixes" applied to satisfy a reviewer.

## Motion A — Running a review (Tier 2+, or any risk surface)

### Step 1 — Set up a fresh-eyes reviewer

Dispatch a subagent (per `delegating-to-agents`) with: the brief (verbatim), the plan's step list + change log, `BASE..HEAD`, and the checklists below. Deliberately exclude your session narrative — a reviewer who heard you reason yourself into the bug will reason the same way past it. No subagent available → review it yourself in a deliberately separate pass: re-read the brief FIRST, then the diff file-by-file, never diff-then-brief (or you'll review what you built instead of what was asked).

### Step 2 — Checklist 1: Intent (against the brief, not the code)

- Every acceptance check implemented? Point at the code per check.
- Anything built that no check asked for? (Scope creep — flag, don't admire. The out-of-scope list is binding.)
- Requirements interpreted differently than the brief's words? Name the divergence.
- Plan change-log entries all reflected? Amendments that didn't land are silent drift.

### Step 3 — Checklist 2: Quality (against the craft)

- **Correctness:** failure paths handled where the happy path is; off-by-ones at boundaries; resources closed; concurrent access where state is shared.
- **Tests:** do they assert real behavior (not mock theater)? Does each new behavior have its failure-path test? Would they catch the change being reverted?
- **Fit:** follows the codebase's existing patterns? New abstractions earn their existence?
- **Risk surfaces:** secrets out of code/logs; injection surfaces parameterized; migrations reversible or staged. **Any risk surface present → a second, independent reviewer pass on those files is mandatory** (different subagent, or the user, per the brief's risk notes).
- Pre-existing mess NOT introduced by this diff: note at most one line; not this task's burden (a recurring one belongs in `retrospecting`).

### Step 4 — Triage and resolve

Findings come back as **Critical** (breaks intent/data/security — blocks the gate), **Important** (fix before landing), **Minor** (note; fix if free, else worklog it). Loop fixes → re-check the specific finding. All Critical/Important resolved → proceed to `verifying-done` (the gate re-runs the proofs; review approval is necessary, not sufficient).

## Motion B — Receiving review feedback

1. **Read all of it before reacting to any of it.** Items often interlock; partial understanding produces wrong fixes.
2. **Verify each claim against the repo before implementing.** Reviewers (human, bot, or the checklists above) are sometimes wrong: check whether the suggestion breaks existing behavior, whether the current code is this way for a reason (git blame, comments, LESSONS.md), whether the "missing" feature is actually called anywhere.
3. Anything unclear → ask about ALL unclear items before implementing ANY item.
4. **Disagree technically when warranted**: state the evidence ("this path is unreachable because X; test Y pins it"), propose the alternative, let the author/user arbitrate. Implementing a change you believe is wrong, silently, is the worst available move.
5. No performative agreement — skip "great catch!"; the fix itself is the acknowledgment. Implement one item at a time, test each, then one evidence entry for the batch.
6. Feedback revealing a brief-level problem (reviewer expected different behavior than the brief specifies) → that's `replanning-on-surprise` (intent-level), not a code fix.

## Rationalization check

| Thought | Reality |
|---|---|
| "I just wrote it, I know it's right" | That's the disqualification, not the credential. Fresh eyes or fresh pass — pick one. |
| "Review will just slow down landing" | Tier 0/1 doesn't require this skill. Tier 2+ earned it by being big enough to be wrong in expensive ways. |
| "The reviewer is a bot, just do what it says" | Verified-then-applied beats applied-then-broken. Every suggestion gets checked against the repo first. |
| "I'll fix the minor stuff later" | Fine — "later" is a worklog line, not a memory. Unwritten laters don't exist. |
