---
name: characterizing-legacy-code
description: Use BEFORE modifying any code that has no tests — "add X to our existing app", "fix this in the legacy module", "refactor this old file" — or when the existing suite doesn't cover the specific code you're about to change. Pin current behavior first so your change can't silently break what works today.
---

# Characterizing Legacy Code

You cannot safely change behavior you haven't pinned. A characterization test asserts what the code *actually does right now* — not what it should do — so that after your change, every diff in behavior is one you chose. This is the brownfield entry ritual: pin, then change.

**Prevents:** failure mode #8 (breaking legacy code — regressions in behavior nobody wrote down but everybody depends on).

## Step 1 — Scope the blast radius

1. Identify exactly what you'll modify (functions/methods/branches).
2. Find every caller (grep/IDE references). The blast radius = the code you change + every behavior reachable from changed lines.
3. Check what tests already exist for any of it: run the suite with coverage on the target file if cheap. Already covered → plain `testing-by-default`. Partially → characterize only the uncovered parts you'll touch.

## Step 2 — Get it in a harness

Try, in order of cheapness:
1. Call it directly from a test (it has injectable/no dependencies) → proceed to Step 3.
2. Call it with light scaffolding (temp dir, in-memory DB, fake clock) → proceed.
3. Can't instantiate it without dragging in the world → STOP, invoke `finding-seams`, return here with a seam.

## Step 3 — Write characterization tests

1. **Ask the code, not your model of it.** Write a test asserting a *wrong but plausible* value: `expect(fee(100)).toEqual(999)`. Run it. The failure message tells you the real answer. Now assert THAT.
   This sounds backwards; it is the point — your guesses about legacy behavior are exactly what can't be trusted.
2. Cover the blast radius, not the module: each input class your change could affect — typical, boundary (empty/zero/null/max), and at least one error path ("what does it currently do with garbage?" — often "returns undefined silently"; pin that too, bugs included).
3. Name them for what they record: `test('charges 2.9% + 30c on amounts over $1 (current behavior)')`.
4. **Pin current behavior even when it's wrong.** Found a bug? Worklog `surprise` entry, tell the user, pin it as-is unless fixing it joins the task's scope (via re-sizing). Silent fixes are scope creep wearing a halo.

Pragmatics: golden-master testing (capture full output, diff later) is legitimate for big string/report outputs. For nondeterminism (time, random, ordering), control the seam or assert invariant properties instead of exact values.

## Step 4 — Verify the net catches

Run the characterization tests. All green against UNCHANGED code (you're asserting reality — red here means scaffolding bugs). Then sanity-check sensitivity: mentally (or actually) flip one behavior you're about to change and confirm a test would fail. A net that can't catch your intended change can't catch your accidents either.

Commit the tests separately, BEFORE the behavior change: `test: characterize <area> before <task-id>`. That commit is the click — the pinned baseline a resume or a revert can stand on.

## Step 5 — Now change, against the net

Proceed test-first (`testing-by-default`) for the NEW behavior. As you work, characterization failures are tripwires:
- **Intended change** → update the assertion, note it (commit message or worklog) — "intended" means the brief/DoD implies it, not that the new value looks fine.
- **Unintended** → you broke something; `debugging-to-root-cause`.

After the task, keep the characterization tests that still document true behavior (they're the module's only spec); delete ones your change made meaningless.

## Stop conditions

- Blast radius keeps growing past your tier's bounds → re-run `sizing-the-task`.
- Behavior is environment-dependent and you can't reproduce locally → say so; don't pin behavior you can't observe. Ask for the environment or descope.

## Rationalization check

| Thought | Reality |
|---|---|
| "I can see what it does by reading it" | Reading tells you intent. Ten years of patches mean behavior diverged. Ask the code. |
| "Characterization tests aren't real tests" | They're the only spec this module has. After your change they're regression tests. |
| "I'll be careful" | Careful isn't a mechanism. The test is. |
| "It's just a small change to the legacy file" | Small changes to unpinned code are how production incidents introduce themselves. |
