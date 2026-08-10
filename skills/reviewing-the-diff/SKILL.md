---
name: reviewing-the-diff
description: Use when Tier 2/3 implementation is complete (before the final verifying-done gate), when any change touches a risk surface (auth, payments, data migration, secrets, concurrency, public APIs), or when review feedback ARRIVES from a human or tool and you're deciding what to do with it. One skill, two motions: running a review, and receiving one.
---

# Reviewing the Diff

Review is a fresh set of eyes on the diff before it becomes permanent. Ratchet runs **two lenses over the same diff, concurrently** — `fresh-eyes-review` (was the right thing built, and is it built well) and `ponytail-review` (should any of it exist at all) — then triages both into one list. Two lenses, not two ceremonial passes: each asks a question the other structurally cannot. Risk, not ritual, decides depth.

This skill orchestrates; the reviewers' own instructions live in their own skills.

**Prevents:** failure mode #2 (scope creep caught at the exit), #1's late echo (intent misses caught before merge), and — in the receiving motion — wrong "fixes" applied to satisfy a reviewer.

## Motion A — Running a review (Tier 2+, or any risk surface)

### Step 1 — Dispatch both reviewers, in parallel

Two subagents (per `delegating-to-agents`), same `BASE..HEAD`, at the same time:

- **`fresh-eyes-review`** — "is this the right thing, built well?" Send the brief **verbatim**, the plan's step list + change log, and `BASE..HEAD`. **Deliberately withhold your session narrative** — a reviewer who heard you reason yourself into the bug will reason the same way straight past it. Hands back **ranked findings**.
- **`ponytail-review`** — "should any of this exist?", the lazy-senior-dev / minimalism lens. Send the diff, the decision ladder, and one line on what the change is for; NOT the brief's full text and NOT your narrative (intent context only softens its YAGNI rung). Hands back a **delete-list**.

Parallel-safe per `delegating-to-agents`: both are read-only over the same diff with disjoint outputs. Both feed Step 2.

**Risk surfaces get a second, independent pass** — auth, payments, migrations, secrets, concurrency, public APIs. It is mandatory on those files and it comes from a different subagent or the user, per the brief's risk notes. `fresh-eyes-review` reports the surfaces it found precisely so you know what to order; a diff you *know* touches one doesn't wait for permission.

No subagent available → run both lenses yourself, as two separate passes in this order, per each skill's own self-pass note. One pass wearing both hats is not two lenses.

### Step 2 — Triage and resolve

Findings come back as **Critical** (breaks intent/data/security — blocks the gate), **Important** (fix before landing), **Minor** (note; fix if free, else worklog it — and when the *reason* for declining it will be re-litigated by someone who never had this session, it wants a durable record instead: `writing-the-issue`). Loop fixes → re-check the specific finding. All Critical/Important resolved → proceed to `verifying-done` (the gate re-runs the proofs; review approval is necessary, not sufficient). A pre-existing-mess note is not this task's burden — but a *recurring* one belongs in `retrospecting`, not in this diff.

**Ponytail's delete-list** merges into this same triage. Default each item to **Minor** — a simplification, not a defect. Escalate to **Important** when the diff introduced a new dependency or a new abstraction that a lower ladder rung (stdlib, native platform, an already-installed dep, or a one-liner) already covered: unnecessary surface area is a cost paid forever. Never **Critical** — removing over-engineering doesn't break intent, data, or security, and the floor (trust-boundary validation, data-loss, security, accessibility) is off ponytail's list by construction. Apply each item only after verifying it against the repo per Motion B — a reviewer that only deletes is as dangerous as one that only adds.

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
