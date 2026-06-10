---
name: debugging-to-root-cause
description: Use when ANYTHING behaves unexpectedly — failing test, bug report, error in logs, build break, flaky behavior, "this worked yesterday" — BEFORE proposing or attempting any fix. Especially use under time pressure, when a fix seems obvious, or when previous fixes haven't stuck. Symptom patches without a named cause are the failure this prevents.
---

# Debugging to Root Cause

A fix you can't explain is a coin flip wearing a confidence interval. The rule: **no fix until the failure is reproduced and a single written hypothesis names the cause.** Three failed fixes means the problem is architectural — the response is a step back, never a fourth patch.

**Prevents:** failure mode #6 (confident wrong fixes — patches that mask, shotgun debugging that thrashes).

## Step 1 — Reproduce before anything

1. Trigger the failure yourself, reliably. Record the exact command/steps in the worklog.
2. Can't reproduce → that IS the current task: vary environment, data, timing; add instrumentation at the failure site. Do not fix what you cannot make happen — you'd never know you fixed it.
3. Reproducible only sometimes → capture the conditions (1-in-5 with what data? under load?). Flakiness is data: it points at timing, ordering, or shared state.

## Step 2 — Gather evidence (cheap to expensive)

1. **Read the entire error.** Stack trace, line numbers, the wording. Errors usually name their cause; agents usually skim them.
2. **Diff against last-known-good.** `git log`/`git diff` since it worked; new dependencies; environment differences. "Worked yesterday" means the cause is in yesterday's diff — usually.
3. **Trace the bad value backward.** From where the symptom appears, walk UP the call chain: what produced this value? what called that with what arguments? Continue until you reach the origin. The fix belongs at the origin, not where the error happened to surface.
4. **Instrument boundaries in multi-component failures** (request → service → DB; CI → build → deploy): log what enters and exits each layer, run once, find the layer where good data turns bad. Then investigate THAT layer only.
5. **Compare against the working sibling.** Similar code that works? List every difference; refuse to assume any "can't matter."

## Step 3 — One hypothesis, in writing

Append to the worklog: `hypothesis: <X causes the failure> because <evidence>`. One at a time. Test it with the **smallest** change that would confirm or refute — often a log line or a narrowed test, not a fix.

- Confirmed → Step 4.
- Refuted → write the next hypothesis. Never stack a second change on an unconfirmed first.
- Can't form one → you lack evidence, return to Step 2; or say "I don't understand X yet" to the user — honest ignorance beats confident thrash.

## Step 4 — Fix at the cause, prove it held

1. **Failing test first** that reproduces the bug (`testing-by-default`, bugfix row — no exceptions). No framework? A one-off script counts, but say why.
2. ONE fix, at the origin of the bad value. No "while I'm here" cleanups riding along.
3. Run the test → passes. Full suite → no new failures. Original symptom → gone (re-run the Step 1 reproduction, not just the new test).
4. Worklog `evidence` entry: cause, fix, proof. Then `verifying-done` owns any completion claim.

## The three-strikes rule

Count your fix attempts. At **three failed fixes**, STOP — the evidence says the problem isn't where you keep looking:
- Each fix surfacing a new symptom elsewhere = coupling/shared state problem, not a local bug.
- "Fixing" requires touching ever more files = the abstraction is wrong.

Write a worklog `surprise` entry, then either `replanning-on-surprise` (mid-plan) or present the architectural finding to the user (standalone). A fourth patch attempt without that conversation is forbidden — it's how codebases accumulate lies.

## When investigation finds "no root cause"

Truly environmental/external (provider outage, cosmic-ray flake)? Legitimate only AFTER Steps 1–3 are exhausted and documented. Then: handle it explicitly (retry, timeout, alert), record what was ruled out, and say the cause is unconfirmed. Most "no root cause" is "stopped looking."

## Rationalization check

| Thought | Reality |
|---|---|
| "It's probably X, quick fix and see" | "And see" is the tell — that's an experiment without a hypothesis, run on production code. Write the hypothesis first; the discipline costs one sentence. |
| "No time for process, this is urgent" | The systematic path is 20 minutes; thrash averages hours and adds new bugs. Urgency argues FOR it. |
| "Just bump the timeout / add a retry / restart it" | Masking the symptom converts a loud bug into a quiet one. Quiet bugs page you at 2am. |
| "I'll fix these three suspicious things at once" | If it works you learned nothing; if it breaks you can't tell which one. One variable at a time. |
| "The error is in module A so the bug is in module A" | The error *surfaced* in A. Trace the value backward before deciding where it was born. |
