---
name: replanning-on-surprise
description: Use the moment reality contradicts the plan or brief — a step's approach doesn't work, an API behaves differently than assumed, tests reveal the design is wrong, a dependency is missing, the estimate was badly off — or when tempted to silently deviate from the plan "because Y is clearly better". Surprises amend the plan in writing; they never fork it silently.
---

# Replanning on Surprise

Plans are hypotheses. A surprise is data that part of the hypothesis failed. Both available errors are expensive: *rigidity* (forcing reality to match a falsified plan) and *drift* (quietly abandoning the plan so it stops describing the work). The protocol converts the surprise into an amended plan with an audit trail, in minutes.

**Prevents:** failure mode #7 (plans that drift from reality) and feeds #9's escape hatch (surprises are how "small" tasks reveal themselves to be large).

## Step 1 — Stop and record (before deciding anything)

Append to `.ratchet/WORKLOG.md`:

```
## [time] <task-id> — surprise
expected: <what the plan/brief assumed>
found:    <what reality showed — include the evidence: error, output, doc link>
```

Writing it BEFORE choosing a response keeps the response honest — surprises narrated after the fact always sound like they were handled well.

## Step 2 — Classify the blast radius

| The surprise invalidates… | Class | Response |
|---|---|---|
| Just this step's *how* (API shape differs, function lives elsewhere) | **Local** | Amend the step in place |
| Several steps / the chosen approach (the library can't do X; the design's data flow is wrong) | **Structural** | Re-plan the remainder |
| The brief's assumptions or an acceptance check (the feature can't work as specified; "what they asked for" ≠ "what's possible") | **Intent-level** | Back to the user — this is not yours to decide |
| The tier itself (risk surface appeared; scope doubled) | **Sizing** | `sizing-the-task` re-size, then continue here at the new tier |

Second surprise on the same task = automatic sizing trigger regardless of class (`sizing-the-task`).

## Step 3 — Amend, with the change on the record

**Local:** edit the step (approach/files/proof), append one line to the plan's `## Change log`: date, step, what changed, why. Resume `executing-with-checkpoints` on the amended step. Total cost: ~2 minutes.

**Structural:**
1. Mark dead steps explicitly in the plan (`~~Step 5~~ invalidated — see change log`), never delete (the audit trail is the point).
2. Salvage audit: which completed steps survive the new approach? Committed work that's now wrong gets reverted deliberately, with a change-log line — not left to rot misleadingly in the tree.
3. Write replacement steps under the same rules as `planning-the-work` (files + proof, no placeholders); re-run its self-check on the remainder (coverage vs. brief, stranger test).
4. Change-log entry summarizing old approach → new approach → why.
5. Update STATE.md (step counts changed; "Next action" points into the new steps). Resume.

**Intent-level:**
1. Update STATE.md: `blocked` (or proceed-on-assumption only if the brief pre-authorized it in `## Assumed without confirmation`).
2. Present to the user: what was assumed, what was found (evidence), 2–3 viable responses with costs, your recommendation. Their answer amends the BRIEF (goal/checks/scope), then cascade: re-validate the plan against the amended brief.
3. No user available + no pre-authorization → stop work, leave STATE.md blocked with the question; an answered question later costs minutes, a wrong guess costs the feature.

## Step 4 — Resume at the right altitude

Re-enter `executing-with-checkpoints` (local/structural) or `writing-the-brief` (intent-level). The surprise is handled when the plan again describes the work — not when the workaround works.

## What this skill is NOT for

- Bugs in code you just wrote that the plan correctly anticipated → just fix them (or `debugging-to-root-cause` if mysterious). A surprise contradicts the *plan*, not your typing.
- "I had a better idea mid-flight" with no contradicting evidence → improvements are legitimate but go through the same Step 3 amendment (usually Local, 2 minutes). What's forbidden is the silent fork, not the better idea.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll just work around it and keep moving" | The workaround makes the plan a lie. Every later reader — including resumed-you — inherits the lie. Two minutes for the change-log line. |
| "Replanning means starting over" | Replanning means amending the REMAINDER. Completed, verified steps are ratchet clicks — they hold. |
| "The user doesn't need to know the design changed" | If it touches the brief's checks, it's their call by definition. Surprises survive contact with users far better than discovered-at-demo divergences. |
| "It's quicker to push through and explain later" | 'Later' arrives at the verification gate, where unexplained scope/approach drift fails the diff check anyway. Pay now, it's cheaper. |
