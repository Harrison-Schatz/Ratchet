---
name: reviewing-the-diff
description: Use when Tier 2/3 implementation is complete (before the final verifying-done gate), when any change touches a risk surface (auth, payments, data migration, secrets, concurrency, public APIs), or when review feedback ARRIVES from a human or tool and you're deciding what to do with it. One skill, two motions: running a review, and receiving one.
---

# Reviewing the Diff

Review is a fresh set of eyes on the diff before it becomes permanent. Ratchet runs **three lenses over the same diff, concurrently** — `fresh-eyes-review` (was the right thing built, and is it built well), `ponytail-review` (should any of it exist at all) and `no-comments-review` (what does it claim about itself that nothing checks) — then triages all three into one list. Three lenses, not three ceremonial passes: each asks a question the others structurally cannot. Risk, not ritual, decides depth.

This skill orchestrates; the reviewers' own instructions live in their own skills.

**Prevents:** failure mode #2 (scope creep caught at the exit), #1's late echo (intent misses caught before merge), and — in the receiving motion — wrong "fixes" applied to satisfy a reviewer.

## Motion A — Running a review (Tier 2+, or any risk surface)

### Step 1 — Dispatch the three reviewers, in parallel

Three subagents (per `delegating-to-agents`), same `BASE..HEAD`, at the same time:

- **`fresh-eyes-review`** — "is this the right thing, built well?" Send the brief **verbatim**, the plan's step list + change log, and `BASE..HEAD`. **Deliberately withhold your session narrative** — a reviewer who heard you reason yourself into the bug will reason the same way straight past it. Hands back **ranked findings**.
- **`ponytail-review`** — "should any of this exist?", the lazy-senior-dev / minimalism lens. Send the diff, the decision ladder, and one line on what the change is for; NOT the brief's full text and NOT your narrative (intent context only softens its YAGNI rung). Hands back a **delete-list**.
- **`no-comments-review`** — "what does it claim about itself that nothing checks?", the comment-as-claim lens. Send `BASE..HEAD` and one line on what the change is for; NOT the brief and NOT your narrative (the author's reasons are exactly what a comment launders into the code). Hands back **verdicts** — KILL, ENCODE, SUPPRESSION, KEEP?. **Skip it by rule** when no hunk in `BASE..HEAD` adds, changes, or sits under a comment and the touched files hold no suppression: write `no-comments-review: skipped — no comment added, changed, or above a changed line in BASE..HEAD; no suppression in the <n> touched files` under the round header in that lens's record, and dispatch nothing.

Tell each reviewer the file to write and the header to open it with —
`.ratchet/review/<task-id>-<lens>.md`, lens = the reviewer's skill name:

```
## [YYYY-MM-DD HH:MM] <round label> (BASE..HEAD)
reviewer: <lens> (subagent | self-pass)
```

Each appends its own findings there before returning. Anything handed back unwritten, you write —
verbatim, before you triage.

Parallel-safe per `delegating-to-agents`: all three read the same diff and write disjoint record files (so an extra pass needs its own lens name); a lens that wrote outside its record is a failed delegation (`delegating-to-agents` Step 3) — one re-dispatch, failure named. All three feed Step 2.

All three briefs name any seam this diff adds with no caller BY DESIGN — consumer in another active task, or the next step — and why the caller cannot ship here; unnamed, it reads as dead code to every lens.

**Risk surfaces get a second, independent pass** — auth, payments, migrations, secrets, concurrency, public APIs. It is mandatory on those files and it comes from a different subagent or the user, per the brief's risk notes. `fresh-eyes-review` reports the surfaces it found precisely so you know what to order; a diff you *know* touches one doesn't wait for permission.

No subagent available → run all three lenses yourself, as separate passes in this order, per each skill's own self-pass note. One pass wearing three hats is not three lenses.

### Step 2 — Triage and resolve

Findings come back as **Critical** (breaks intent/data/security — blocks the gate), **Important** (fix before landing), **Minor** (note; fix if free, else worklog it — and when the *reason* for declining it will be re-litigated by someone who never had this session, it wants a durable record instead: `writing-the-issue`). Loop fixes → re-check the specific finding. All Critical/Important resolved → proceed to `verifying-done` (the gate re-runs the proofs; review approval is necessary, not sufficient). A pre-existing-mess note is not this task's burden — but a *recurring* one belongs in `retrospecting`, not in this diff.

**Ponytail's delete-list** merges into this same triage. Default each item to **Minor** — a simplification, not a defect. Escalate to **Important** when the diff introduced a new dependency or a new abstraction that a lower ladder rung (stdlib, native platform, an already-installed dep, or a one-liner) already covered: unnecessary surface area is a cost paid forever. Never **Critical** — removing over-engineering doesn't break intent, data, or security, and the floor (trust-boundary validation, data-loss, security, accessibility) is off ponytail's list by construction. Apply each item only after verifying it against the repo per Motion B — a reviewer that only deletes is as dangerous as one that only adds.

**No-comments' verdicts** merge into this same triage; the lens ranks nothing, so the severity is yours. **KILL** → **Minor**. **ENCODE** → **Important** when the constraint guards correctness, data, or a risk surface (an unenforced invariant is the failure-path test `verifying-done` already demands), else **Minor**; where the lens says the code may be the wrong side, write the deciding test now, and if it fails the finding is **Critical** under fresh-eyes' definition. Resolve an ENCODE at the lowest rung in scope, then delete the prose: a test or assertion inside the diff's own files is a review fix like any other, per `testing-by-default`; an encoding that changes a type other callers see, adds a dependency, or adds a CI check is offered to the user with its rung and cost; declined, deferred, or no user this session → the prose goes and at most the record reference stays on the line (`writing-the-issue` — the record carries the constraint and the proposed rung). An encoding that needs a shape beyond one test, type, or assertion is a plan amendment (`replanning-on-surprise`), not a review fix. **SUPPRESSION** → **Important**. **KEEP?** stands only with its checkable pointer, which you verify. Ambiguity resolves toward deletion, for KILL and KEEP? alike, and neither is ever **Critical** — a comment's absence breaks nothing; what a comment *hides* may, and that is the deciding test above.

**Every finding gets a disposition, in the record.** This is your write, not the reviewer's:
as triage resolves each item, insert one line beneath it, editing nothing else in the file:

```
  → disposition: FIXED @ <commit> | DECLINED — <evidence> | ISSUE #<n> | DEFERRED — <worklog ref>
```

## Motion B — Receiving review feedback

1. **Read all of it before reacting to any of it** — every lens record, when more than one ran: one lens's replacement text can preserve the exact claim another falsified. Items often interlock; partial understanding produces wrong fixes.
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
| "The findings are in the transcript" | The transcript is not disk, and it ends with this session. |
