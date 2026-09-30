<!-- Template for .ratchet/review/<task-id>-<lens>.md. Format owned by `reviewing-the-diff` (Step 1, round header; Step 2, disposition), `fresh-eyes-review` (Output) and `ponytail-review` (Output).
     If this file and those skills disagree, the skill wins.
     Delete this comment when you copy. -->
<!-- keep the finding block for YOUR lens, delete the other. Per-item line written at triage (reviewing-the-diff > Step 2):
     → disposition: FIXED @ <commit> | DECLINED — <evidence> | ISSUE #<n> | DEFERRED — <worklog ref> -->

## [<YYYY-MM-DD HH:MM>] <round label> (<BASE>..<HEAD>)
reviewer: <lens> (<subagent | self-pass>)

<!-- fresh-eyes-review -->
- <Critical | Important | Minor> — <file:line> — <the defect in one sentence>
  why: <the concrete failure: inputs or state → wrong result. Not "this is fragile">
  fix: <the smallest change that resolves it, or "unclear — needs the author">

risk surfaces: <every surface the diff touches, finding or not> | none

<!-- ponytail-review -->
- <file:line> — <what to remove or replace>
  rung: <which ladder rung, 1–6>
  instead: <the concrete replacement: delete entirely | stdlib X | native Y | dep Z | one-liner>
  why: <one line — what this buys, and the upgrade path if the shortcut ever needs to grow>
