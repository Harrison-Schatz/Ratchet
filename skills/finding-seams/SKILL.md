---
name: finding-seams
description: Use when code can't be tested because its dependencies are baked in — constructors that hit the network/DB/filesystem/clock, global singletons, static calls, "I'd have to mock everything", "I can't instantiate this in a test" — typically discovered from characterizing-legacy-code Step 2 or testing-by-default's stop condition. Creates the smallest safe opening to get code under test.
---

# Finding Seams

A seam is a place where you can change behavior in a test without editing the code under test — swap the database, freeze the clock, fake the API. Legacy code resists testing because it has none. Your job: create the *smallest* seam that lets the test in, using transformations safe enough to perform without a test net (because, circularly, there isn't one yet).

**Prevents:** failure mode #8 (breaking legacy code) at its root — "couldn't test it" is the parent of "didn't test it".

## Step 1 — Name the blocker

Write one sentence: "I can't test `<unit>` because `<dependency>` is `<hardwired how>`." Common shapes: constructor does real work (connects, reads config); static/global access (`DB.query`, `new Date()`, singletons); concrete instantiation inside the method; the method is 400 lines and the logic you need is in the middle of it.

## Step 2 — Pick the cheapest seam that answers the blocker

Work DOWN this list; stop at the first that fits. Cost order matters — each later option edits more untested code.

1. **Parameter seam** — the dependency (or just the *value* it produces) becomes a parameter with a default preserving current behavior:
   `processOrders(orders, now = new Date())`. Callers unchanged. The 80% answer for clocks, config, randomness.
2. **Extract-the-logic seam** — pure logic tangled with I/O: extract the decision into a pure function, leave the I/O where it was. `if (user.lastLogin < cutoff && !user.vip)` → `isExpired(user, cutoff)`. Test the logic exhaustively; the I/O wrapper keeps its one thin path.
3. **Constructor seam** — dependency created in the constructor: accept it as an optional constructor arg, defaulting to the current creation. Tests inject; production unchanged.
4. **Extract-and-override seam** — dependency woven through a class you can't restructure: wrap the access in a protected method; the test subclasses and overrides it. Ugly, deliberately temporary, beats a rewrite you can't afford.
5. **Sprout seam** — the host function is too hostile to touch at all: write your NEW code as a fresh, fully-tested function/class and have the legacy code make one call into it. The legacy mess stays unimproved but *unworsened*; your change is 100% tested. Often the right answer for "add X to a 400-line function."
6. **Wrap seam (adapter)** — a third-party/global API used everywhere: introduce a thin interface you own, used at the one place you're changing. Don't retrofit the whole codebase — that's a separate task someone can choose later.

Deeper treatment of each, with before/after code: `references/seam-catalog.md` — read it when the shape you're facing isn't obviously one of the above.

## Step 3 — Cut the seam under safe-edit rules

You are editing untested code; the transformations must be mechanical:
- Signature-preserving by default (new params get defaults; new ctor args optional).
- IDE/automated refactors over hand edits where available.
- **No behavior changes whatsoever in the seam commit** — even a log line you "fixed in passing" contaminates the baseline you're about to pin.
- Compile/run after each micro-step (lean on the type checker).
- Seam commit separate and labeled: `refactor: add clock seam to OrderProcessor (no behavior change)`.

## Step 4 — Use it, then reassess

Return to `characterizing-legacy-code` Step 3 and pin behavior through the new seam. After the task lands, ONE worklog line on whether the seam should graduate (e.g., "extract-and-override in Billing is temporary; proper DI worth a task") — logged for a human to choose, not silently done (`retrospecting` will sweep it into LESSONS.md if it recurs).

## Stop conditions

- Every seam option requires restructuring beyond your tier's bounds → re-run `sizing-the-task`; tell the user the testability cost honestly ("getting this safely under test is most of the work").
- The dependency is genuinely unfakeable locally (proprietary hardware, prod-only service) → seam at the *boundary you can control*, integration-test what's inside it, and record the unverified gap explicitly in the worklog — `verifying-done` will cap claims accordingly.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll just mock the module loader / monkeypatch globals" | Patching infrastructure couples the test to internals and rots fast. A parameter is smaller, honest, and survives refactoring. |
| "While I'm in here I'll clean this whole class up" | The seam commit must stay mechanical — it's the only thing making untested edits safe. Cleanup is a separate, later, pinned task. |
| "Refactoring without tests is forbidden, so I'm stuck" | These specific signature-preserving moves exist precisely for the no-tests bootstrap. That's why the list is short and mechanical. |
