<!-- Template for .ratchet/worklog/<task-id>.md. Format owned by `keeping-state` (worklog/<task-id>.md — append-only per-task entries); entry shapes from `sizing-the-task` (Step 3, sizing), `verifying-done` (The gate procedure, step 5, gate evidence), `landing-the-change` (Step 3, done) and `retrospecting` (Step 5, retro).
     If this file and those skills disagree, the skill wins.
     Delete this comment when you copy. -->
# Worklog: <task-id>   <!-- optional: title line -->

<!-- entry shapes, not a sequence: copy the one you need per event, append it, keep no unused stub -->

## [<YYYY-MM-DD HH:MM>] <task-id> — sizing
Tier <1-3>. Goal: <one sentence>.
Done when: <1-3 observable checks>
Out of scope: <what you are explicitly NOT doing>

## [<YYYY-MM-DD HH:MM>] <task-id> — <decision | surprise | blocked>
<1-4 lines: what + why; link files rather than pasting them>

## [<YYYY-MM-DD HH:MM>] <task-id> — escalation
Tier <old> → Tier <new>: <the trigger that fired>

## [<YYYY-MM-DD HH:MM>] <task-id> — evidence
ran: <command> → <verbatim result summary: counts, exit code>

## [<YYYY-MM-DD HH:MM>] <task-id> — evidence (gate, T<n>)
ran: <test command> → <verbatim result summary: N passed, N failed, exit code>
ran: <lint or build command> → <verbatim result summary>
scope: <N files, all within plan or Done-when>; plus <undeclared-but-defensible edit> (declared)
checks: <brief acceptance check or Done-when check → the evidence that shows it, one per check>

## [<YYYY-MM-DD HH:MM>] <task-id> — done
<what landed: merge SHA or PR URL>; evidence: <time of the gate evidence entry | Tier 0: ran: <command> → <verbatim result>>

## [<YYYY-MM-DD HH:MM>] <task-id> — retro
<lessons added / merged / pruned (L<n>), or "no lessons — clean task">
