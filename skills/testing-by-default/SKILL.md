---
name: testing-by-default
description: Use BEFORE writing any implementation code — new feature, bugfix, refactor, behavior change — at any tier above 0, and whenever deciding "how should I test this?" or feeling tempted to "add tests after". Picks the right test discipline for the territory: test-first, characterize-first, or spike-then-stabilize.
---

# Testing by Default

Untested code is a claim; tested code is a fact. The default is test-first — but "default" means the burden of proof is on deviating, and the legal deviations are *defined disciplines*, not vibes. Pick the discipline by the territory you're in, declare it (plan step or worklog line), and follow it exactly.

**Prevents:** failure mode #4 (untested happy path) and #8's cousin (regressions from changing code whose behavior was never pinned).

## Step 1 — Pick the discipline by territory

| Territory | Discipline |
|---|---|
| New code, or existing code WITH tests and usable seams | **Test-first** (below) |
| Existing code WITHOUT tests that you must change | **Characterize-first** → invoke `characterizing-legacy-code` BEFORE touching it |
| Genuine unknowns — unfamiliar API, feasibility question, "will this even work?" | **Spike-then-stabilize** (below; for state-model or UI questions, `prototyping-to-decide` is the structured playbook) |
| Bugfix, any territory | **Test-first, always** — reproduce as a failing test before fixing; no exceptions, because a bug is by definition a behavior you can demonstrate |
| Not provable by automated test (visual polish, copy, perf feel) | **Declared manual proof**: state in the worklog WHAT you will observe to verify, then observe it at the gate. Unprovable ≠ unverified. |

Declare the choice where the work is recorded. An undeclared discipline is a missing one.

## Test-first (the default)

1. **RED** — write ONE minimal test for the next behavior. Run it. **Watch it fail, for the expected reason.** Passes immediately → it tests existing behavior; fix the test. Errors (typo, import) → fix and re-run until it *fails properly*.
2. **GREEN** — write the minimum code to pass. Run it. Watch it pass. Other tests must stay green.
3. **REFACTOR** — clean up names/duplication with everything green. No new behavior.
4. Repeat per behavior. Commit at green points.

Wrote implementation before the test? The cheap honest fix: stash it (`git stash`), write the test, watch it fail against the pre-change code, unstash, watch it pass. The test has now proven it detects the change. (A test written after and never seen failing proves nothing — that's the actual sin, not the keystroke order.)

**Required test classes per change:** the happy path, at least one failure path (bad input, dependency error), and the edge you'd least like to debug at 2am (empty, zero, concurrent, unicode — pick what bites this domain). Ask "what breaks this?" and write that test. `verifying-done` checks the failure path exists at T1+.

## Spike-then-stabilize (exploration)

When the unknown is a state model, data shape, or UI design, invoke `prototyping-to-decide` — it runs this same discipline with a concrete playbook (interactive logic harness, or N structurally-different UI variants). Otherwise:

1. Declare the spike in the worklog: question, scope, timebox.
2. Hack freely — no tests, no quality bar. The output is an *answer*, not code.
3. At the timebox: record the answer in the worklog.
4. **The spike does not ship.** Re-implement test-first. Keeping spike code is legal only if you then write its tests, watch each fail via stash-revert, and pass review as if fresh — usually slower than rewriting. Default: delete.

## Anti-patterns (iron rules)

- **Never assert on mock behavior.** A test that proves your mock works proves nothing. Mock only what you can't run (network, clock, money), mirror the real API completely, and prefer integration tests when mocks grow complex.
- **Never add production methods only tests use.** Test through the public surface; needing a backdoor means the design or the test is wrong.
- **Never weaken a test to make it pass.** A failing assertion is information about the code, not an inconvenience in the test.
- **Never delete/skip a failing test to clear a gate.** Skips require a worklog `decision` entry with a reason and a revisit trigger.

## Stop conditions

- Test is hard to write because the code has no seam → `finding-seams`, don't force a 40-line mock jungle.
- Test setup needs half the app running → wrong test level; test a smaller unit or go full integration, whichever matches the behavior.
- You can't name what the test should assert → you don't understand the requirement; back to the brief/DoD, not forward into code.

## Rationalization check

| Thought | Reality |
|---|---|
| "Too simple to test" | Simple = 30-second test. If it's too trivial for a test, why does the change exist? |
| "I'll add tests after this works" | Tests-after verify what you built; tests-first verify what was wanted. Different questions; only one catches the misunderstanding. |
| "Mocking everything is the only way" | That's the code telling you it has no seams. `finding-seams`. |
| "This is exploratory" | Fine — declare a spike with a timebox. Undeclared exploration that quietly ships IS the failure. |
| "The deadline is tight" | The failure-path test is minutes. The 2am production debug is hours. You're not trading speed for rigor; you're choosing when to pay. |
