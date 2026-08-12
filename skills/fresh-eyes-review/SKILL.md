---
name: fresh-eyes-review
description: Use to review a diff against the brief that asked for it and hand back ranked findings — dispatched in parallel with ponytail-review by reviewing-the-diff, or standalone when the user says "review this", "fresh eyes on this", "did I build what was asked", or when a change touches a risk surface (auth, payments, migrations, secrets, concurrency, public APIs). The reviewer who never heard the author reason.
---

# Fresh Eyes Review

You are reviewing a diff you did not write, and you were deliberately not told how the author got there. That absence is the whole point: a reviewer who hears someone reason their way into a bug reasons the same way straight past it. Two questions, in this order — **was the right thing built**, then **is it built well**. Findings only, ranked, each one checkable against the repo.

**Prevents:** failure mode #1's late echo (intent misses caught before merge) and #2 at the exit (scope creep flagged instead of admired) — the catches `reviewing-the-diff` promises are made here.

## What you get, and what you must not ask for

The brief verbatim, the plan's step list and change log, and `BASE..HEAD`. Nothing else. If a session narrative reaches you anyway, ignore it — and if part of the diff seems to *need* that narrative to make sense, that is itself a finding: code a reviewer cannot follow is code the next maintainer cannot follow.

**Read the brief first, then the diff.** Never the reverse. Diff-then-brief reviews what was built instead of what was asked, which is the exact failure this skill exists to catch.

## Checklist 1 — Intent (against the brief, not the code)

- Every acceptance check implemented? Point at the code, per check.
- Anything built that no check asked for? Scope creep — flag it, don't admire it. The out-of-scope list is binding.
- Any requirement interpreted differently from the brief's words? Name the divergence; never reconcile it silently on the author's behalf.
- Every plan change-log entry reflected in the diff? An amendment that didn't land is silent drift.

## Checklist 2 — Quality (against the craft)

- **Correctness:** failure paths handled wherever the happy path is; off-by-ones at boundaries; resources closed; concurrent access wherever state is shared.
- **Tests:** do they assert real behavior or mock theater? Does each new behavior have its failure-path test? Would they actually fail if the change were reverted?
- **Fit:** does it follow the patterns already in this codebase? Do new abstractions earn their existence?
- **Risk surfaces:** secrets out of code and logs; injection surfaces parameterized; migrations reversible or staged. **Name every risk surface this diff touches, explicitly, even where you found nothing wrong** — the dispatcher owes those files a second independent pass and cannot order one for a surface you never named.
- **Pre-existing mess** not introduced by this diff: at most one line, labelled as pre-existing. It is not this task's burden.

## Output

Findings ONLY. No preamble, no "solid work overall", no summary of what the change does — whoever dispatched you has the diff. Each item:

```
- <severity> — <file:line> — <the defect in one sentence>
  why: <the concrete failure: inputs or state → wrong result. Not "this is fragile">
  fix: <the smallest change that resolves it, or "unclear — needs the author">
```

Then one line listing the risk surfaces the diff touches, or `risk surfaces: none`.

Severity, and hold the boundaries:

- **Critical** — breaks intent, data, or security. Blocks the gate.
- **Important** — must be fixed before landing, but nothing is lost or exposed.
- **Minor** — worth saying once: a name, a missing test on a trivial path, a style break from the surrounding code.

"I would have done it differently" is not a finding at any severity. If the diff is clean, say so in one line and return an empty list — a review that manufactures findings to look thorough costs more than it saves, because every false finding spends the author's trust and their afternoon.

Whoever dispatched you records this output **verbatim** into the project's review record, where
it outlives the session (`reviewing-the-diff` Step 2). So write findings that stand alone — file,
line, and the concrete failure — with no reliance on anything said in chat. You write nothing
yourself; read-only stands.

## Rules

- **Every finding names a concrete failure**, not a feeling. "This could be racy" is not a finding; "two requests reaching `claim()` between the SELECT and the UPDATE both succeed, so the row is claimed twice" is.
- **Verify before you claim.** Read around the hunk, not just the hunk: a "missing" guard often sits three lines above the context window, and an "unused" function usually has the one caller you didn't grep for.
- **You produce findings; you do not fix.** Read-only. Never edit a file.
- **Review the diff, not the author.** No praise, no blame, no theories about why they did it that way.
- **Intent outranks quality** when you must choose where to spend attention. Perfectly built code answering the wrong brief is a total loss; a slightly ugly implementation of the right thing is not.

## When you are the same session that wrote the diff

No subagent available? Then this is a deliberately separate pass, and the discipline gets *harder*, not softer: re-read the brief in full before you open the diff, go file-by-file, and write every finding down before you decide which to fix — into the review record, not a scratch list; your own pass gets its file exactly as a subagent's would. Deciding as you read is how findings quietly become "actually that's fine". "I just wrote it, I know it's right" is the disqualification, not the credential.
