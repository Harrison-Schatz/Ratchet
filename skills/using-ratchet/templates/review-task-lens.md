<!-- Template for .ratchet/review/<task-id>-<lens>.md. Format owned by `reviewing-the-diff` (Step 1, round header; Step 2, disposition), `fresh-eyes-review` (Output), `ponytail-review` (Output) and `no-comments-review` (Output).
     If this file and those skills disagree, the skill wins.
     Delete this comment when you copy. -->
<!-- keep the finding block for YOUR lens, delete the others. Per-item line written at triage (reviewing-the-diff > Step 2):
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

<!-- no-comments-review -->
- <KILL | ENCODE | SUPPRESSION | KEEP?> — <file:line> — <the comment, quoted>
  claims: <what the comment asserts about the code, as one checkable sentence>
  checked: <what you ran or read — the blame commit, the grep for the referent, the rule's docs — and what it showed>
  instead: <KILL: delete | ENCODE: rung N — the type, test, assertion or lint that holds it | SUPPRESSION: the scoped, reasoned, expiring form, or "fix the error" | KEEP?: the pointer that would earn it>

suppressions in touched files: <n> (<n> unscoped, <n> unreasoned, <n> on correctness rules) | none
