---
name: prototyping-to-decide
description: Use when a design choice can't be settled on paper — during writing-the-brief when 2-3 approaches need evidence to choose between, when a data model or state machine "looks right but feels uncertain", for UI/UX questions no written acceptance check can settle, or when the user says "prototype this", "let me play with it", "mock it up", "try a few designs". Builds throwaway code that answers ONE question, under declared spike rules.
---

# Prototyping to Decide

A prototype is **throwaway code that answers a question**. It is the structured form of `testing-by-default`'s spike-then-stabilize discipline: cheap evidence for a decision that would otherwise be a guess baked into a brief or plan. The question decides the shape; the answer is the only deliverable.

**Prevents:** failure mode #1 (building the wrong thing — the expensive build starts after the cheap evidence, not before) and #7 (plan drift — approach choices settled by prototype don't collapse at step 7 of 10).

## Step 0 — Declare the spike (before writing any code)

Append to `.ratchet/WORKLOG.md`:

```
## [time] <task-id> — decision (spike opened)
question: <ONE question this prototype answers>
shape: logic | ui
timebox: <e.g. 90 min>
```

An undeclared prototype is just untested code with an excuse. The declaration is what makes the no-tests/no-polish rules legal.

## Step 1 — Pick the shape by the question

- **"Does this logic / state model / data shape hold up?"** → read `references/logic-prototypes.md`. A tiny interactive terminal app drives a *pure* state module through the cases that are hard to reason about on paper.
- **"What should this look like?"** → read `references/ui-prototypes.md`. Several **structurally different** UI variants on one route, switchable via `?variant=` and a floating bar; prefer mounting inside an existing page so variants are judged against real surroundings, not a vacuum.

Wrong shape wastes the whole prototype. Genuinely ambiguous and the user unreachable → default by the surrounding code (backend module → logic; page/component → ui) and state the assumption in the spike declaration.

## Rules for both shapes

1. **One question.** A second question is a second (declared) spike.
2. **Throwaway from day one, and marked.** Name and place it so a casual reader sees "prototype", following the project's existing conventions — no new top-level structure.
3. **One command to run**, wired into the project's existing task runner. The user must be able to start it without thinking.
4. **No persistence** unless persistence IS the question (then a scratch store with a "PROTOTYPE — wipe me" name).
5. **Skip the polish.** No tests, no error handling beyond runnable, no abstractions, no "support X later" — these are spike rules, already declared in Step 0.
6. **Surface the state.** Every action (logic) or variant switch (ui) shows the full relevant state — the prototype exists to make the idea's bugs visible.
7. **Keep the salvageable part pure** (logic shape): the reducer/state machine lives behind a clean interface with zero I/O or terminal code; the TUI shell around it is the throwaway part.

## Step 2 — Hand it over and watch

Give the user the run command (or URL + variant keys). The valuable moments are "wait, that shouldn't be possible" and "I want the header from B with the sidebar from C" — those are defects in the *idea*, caught at prototype price.

## Step 3 — Capture the answer in the worklog (not a stray file)

When the question is answered, append:

```
## [time] <task-id> — decision (spike closed)
question: <same question>
answer: <what was learned, in 1-3 sentences>
evidence: <what demonstrated it — "user drove the TUI through the double-cancel case", "variant B chosen, C's sidebar grafted">
```

Feed the answer where it belongs: the brief's **Approach** section (if spiking during `writing-the-brief`), the plan's change log (if spiking during a replan), or the Tier 1 DoD. If the user is AFK, the entry stands as the record; close it when they weigh in.

## Step 4 — Delete or graduate

- **Default: delete.** The answer is captured; the code served its purpose.
- **Graduating the pure logic module** is legal ONLY through `testing-by-default`'s spike-then-stabilize rules: write its tests, watch each fail against the pre-module state (stash-revert), pass review as if fresh. The TUI shell and UI variant scaffolding never graduate — losing variants and switcher are deleted when the winner is folded in (rewritten properly, not promoted as-is).
- **Enforcement is structural:** leftover prototype code in the final diff fails `verifying-done`'s scope check — it traces to no acceptance check and sits on no out-of-scope list. Delete it before the gate, not because of the gate.

## Rationalization check

| Thought | Reality |
|---|---|
| "The prototype works, I'll just ship it" | It was written under no-tests/no-polish rules you declared yourself. Graduate the pure module through tests, or rewrite. |
| "I'll skip the declaration, it's just a quick experiment" | The declaration is two lines and it's what separates a spike from undisciplined code. Undeclared exploration that quietly ships is the failure this prevents. |
| "Three similar variants are faster to make" | Three near-identical drafts answer nothing. Structurally different or it's wallpaper. |
| "I'll keep the prototype around in case we need it" | The ANSWER is kept — in the worklog. Rotting prototype code misleads the next reader. |
