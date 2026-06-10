---
name: using-ratchet
description: Load at the start of EVERY session and before responding to ANY request that could change files, fix something, build something, or answer "is it done?" — including requests that look trivial ("fix this typo", "quick question about a bug"). Also load whenever a .ratchet/ directory exists in the repo, whenever you are unsure which Ratchet skill applies, or whenever you are about to act without having sized the task.
---

# Using Ratchet

Ratchet is the operating methodology for this project. It exists because agent work fails in known ways: building the wrong thing, silent scope creep, completion theater, lost context between sessions, confident wrong fixes, plans that drift from reality. Every Ratchet skill defeats one of those failure modes. This skill tells you which one fires when.

**Prevents:** skills never firing at all (the meta-failure). If you don't route, nothing downstream works.

## Step 0 — Orient (always, before anything else)

1. Check for `.ratchet/STATE.md`.
   - **Exists and shows a task in flight** → this is a resume, not a new task. Invoke `resuming-work`. STOP here; that skill takes over.
   - **Exists and idle, or doesn't exist** → continue.
2. Check for `.ratchet/LESSONS.md`. If it exists, read it now — it contains project-specific rules earned from past failures. They override your defaults.
3. If `.ratchet/` doesn't exist and the task will be Tier 1+, create it when sizing (see `keeping-state` for the layout).

## Step 1 — Route the request

| The user's message is… | You invoke | Examples |
|---|---|---|
| A request to change anything (feature, fix, refactor, config, docs) | `sizing-the-task` — ALWAYS first | "add OAuth", "fix this typo", "rename that flag" |
| A bug report, failing test, or "this behaves weirdly" | `sizing-the-task`, then `debugging-to-root-cause` | "tests broke", "500 on login" |
| "Continue", "where were we", a fresh session on existing work | `resuming-work` | — |
| "Is it done?", "ship it", "looks good, merge" | `verifying-done`, then `landing-the-change` | — |
| Review feedback arriving | `reviewing-the-diff` (Receiving section) | PR comments |
| A pure question with no change implied | Answer it. No ceremony. | "what does this function do?" |

Routing rule: **sizing comes before clarifying questions.** Sizing tells you whether questions are even needed (Tier 0–1 shouldn't need any).

## Step 2 — Follow the spine

```
ORIENT → SIZE → BUILD → VERIFY → RECORD
```

The tier set by `sizing-the-task` determines the BUILD path:

- **Tier 0 (patch):** implement directly → `verifying-done` (T0 evidence) → one worklog line. Done. No brief, no plan, no questions.
- **Tier 1 (task):** write Definition of Done in worklog → implement per `testing-by-default` → `verifying-done` (T1) → `landing-the-change`.
- **Tier 2 (feature):** `writing-the-brief` → `planning-the-work` → `executing-with-checkpoints` → `verifying-done` (T2) → `landing-the-change` → `retrospecting`.
- **Tier 3 (project):** as Tier 2, plus decomposition into milestones inside the brief; each milestone runs the Tier 2 loop.

## Cross-cutting skills — fire on their trigger, at any phase

| Trigger | Skill |
|---|---|
| You learned something a fresh session would need; phase boundary crossed | `keeping-state` |
| Reality contradicted the plan or brief; estimate blown; second surprise | `replanning-on-surprise` |
| About to modify code that has no tests | `characterizing-legacy-code` |
| Can't get legacy code under test — dependencies block you | `finding-seams` |
| About to write implementation code | `testing-by-default` |
| Anything broken or unexpected | `debugging-to-root-cause` |
| A design choice can't be settled on paper; "prototype this", "let me play with it", "try a few designs" | `prototyping-to-decide` |
| Work splittable across agents, or context running long | `delegating-to-agents` |
| Tier 2+ implementation complete | `reviewing-the-diff` |
| Landing a Tier 2/3 change; diff touches public surface docs describe; "update the docs" | `syncing-the-docs` |
| About to say "done", "fixed", "passing", or any synonym | `verifying-done` |

## The three structural rules

These hold at every tier, with no exceptions to remember because they're cheap:

1. **No completion claim without an evidence line in the worklog.** If `verifying-done` hasn't produced one, the claim is blocked.
2. **STATE.md is updated at every phase boundary.** A session that dies right now should be resumable from disk alone.
3. **Never edit the default branch directly past Tier 0.** Branch first; the harness's native isolation tools are fine.

## If you're tempted to skip routing

The proportionality judgment you're about to make ("this is too small for process") is exactly what `sizing-the-task` formalizes — it takes under a minute and its Tier 0 answer is "just do it." Skipping sizing doesn't save the minute; it discards the record that lets anyone trust the shortcut was legitimate.
