---
name: writing-the-brief
description: Use when a task has been sized Tier 2 or 3, when the user asks to "build", "add", "create", or "redesign" something with real ambiguity, or when you catch yourself guessing what the user actually wants. The brief is the contract that prevents building the wrong thing — do not start planning or coding a Tier 2+ task without one.
---

# Writing the Brief

Turn an ambiguous request into a short, confirmed statement of intent with observable acceptance checks. The brief answers *what and why*; the plan (later) answers *how*.

**Prevents:** failure mode #1 (building the wrong thing) and #2 (scope creep — the out-of-scope list is binding).

## Step 1 — Look before asking

Spend a few minutes in the codebase first: relevant modules, existing patterns, recent commits, anything in `.ratchet/LESSONS.md` that touches this area. Questions you could have answered yourself burn the user's patience.

## Step 2 — Ask what you genuinely cannot infer

- One question per message; prefer concrete options over open-ended ("A: store tokens in session, B: in DB — B survives restarts, costs a table. I'd pick B").
- Ask only about: purpose, hard constraints, success criteria, and any risk-surface decisions (auth, data, money). Skip what the codebase or conventions already answer.
- **Never ask the user to answer a UI/UX question in words.** Layout, interaction flow, "does this feel right," visual hierarchy — these cannot be answered accurately in text by either party. If a question on your list is really a UI/UX question, don't ask it; route it to `prototyping-to-decide` and replace the question with a one-line note: "UX question — will resolve via prototype, not Q&A." The user reacts to something rendered, never to a description of it.
- 2–5 questions is normal. Zero is fine if the request was detailed. Fifteen means the task is really Tier 3 — say so and decompose.

## Step 3 — Propose approaches when there's a real choice

If two or more genuinely different designs exist, present 2–3 with trade-offs and your recommendation first. If there's only one sensible approach, don't manufacture alternatives — say what you'll do.

Decisions split into two kinds:

- **Paper-decidable** — data models, API shapes, storage choices, algorithms. Argue these in the Approach section with trade-offs. If one still "looks right but feels uncertain," invoke `prototyping-to-decide` to settle it with evidence.
- **Not paper-decidable — all UI/UX decisions.** Layout, navigation, interaction patterns, anything a user will see or touch. These are never decided on paper, no matter how confident the written argument feels — a prose description of a UI is a guess wearing a suit. Invoke `prototyping-to-decide` by default: a declared, timeboxed throwaway the user can react to, producing evidence for the Approach section. The only exemptions are trivial conformance cases (the design system or an existing screen already dictates the answer) — and then you're not deciding, you're following.

## Step 4 — Write the brief

Save to `.ratchet/briefs/<task-id>-brief.md`:

```markdown
# Brief: <title>            <!-- task-id: YYYY-MM-DD-slug -->

## Goal
<2-4 sentences: what exists when this is done, and why it's wanted>

## Acceptance checks
<numbered, each one OBSERVABLE — a command, a user action with visible result,
 a test that passes. "Works correctly" is not a check. 3-8 items.>
1. `POST /login` with valid Google token returns a session cookie; integration test proves it.
2. ...

## Out of scope
<explicit non-goals. This list is what verifying-done diffs against.>

## Approach
<the chosen approach in 3-6 sentences; alternatives rejected and why, one line each>

## Risk notes
<which risk surfaces this touches and what that obligates (e.g. "auth: second reviewer at gate")>

## Open questions
<anything deferred, with what unblocks it>
```

For **Tier 3**, add a `## Milestones` section decomposing into independently-shippable Tier 2 features, each one line: goal + rough acceptance check. Each milestone later gets its own brief-lite (goal + checks can be extracted from this section).

## Step 5 — Confirm

Show the user the brief (inline summary plus file path). Ask for approval or edits. **This is a blocking gate for Tier 2/3** — the one place Ratchet always waits, because intent is the one thing you cannot verify from the repo.

If the user is unavailable (autonomous run): record your interpretation and assumptions in the brief under `## Assumed without confirmation`, proceed, and treat any later evidence contradicting an assumption as a mandatory `replanning-on-surprise` stop.

## Step 6 — Hand off

On approval: update `.ratchet/STATE.md` (phase: `planning`), then invoke `planning-the-work`. The ONLY skill that follows an approved brief is `planning-the-work`.

## Stop conditions

- User rejects the framing → revise, re-confirm. Two rejections mean you misunderstand the domain: go back to Step 2 with different questions.
- Brief reveals the task is really Tier 1 (no ambiguity, one subsystem) → de-escalate via `sizing-the-task`, convert the acceptance checks into the worklog Definition of Done, skip the plan.

## Rationalization check


| Thought                              | Reality                                                                                                |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| "The request was clear enough"       | Then the brief takes ten minutes and the user confirms in one word. Cheap insurance on multi-day work. |
| "I'll clarify as I go"               | Mid-build clarification means rework. The checks you can't write now are the rework you'll do later.   |
| "More features would make it better" | Write them in Out of scope. That's where they belong until asked for.                                  |


