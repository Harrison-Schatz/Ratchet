<!-- Template for .ratchet/worklog/<task-id>.md. Format owned by `keeping-state` (worklog/<task-id>.md — append-only per-task entries), with `sizing-the-task` (Step 3) for the sizing entry and `verifying-done` (The gate procedure, step 5) for the gate evidence entry.
     If this file and those skills disagree, the skill wins.
     Delete this comment when you copy. -->
# Worklog: <task-id>   <!-- optional: title line; id form: keeping-state > The layout -->

<!-- one entry per event, appended; entry types and length: rule: keeping-state > worklog/<task-id>.md; which entry when: rule: keeping-state > When to write -->

## [<YYYY-MM-DD HH:MM>] <task-id> — sizing
Tier <1-3>. Goal: <one sentence>.
Done when: <1-3 observable checks>
Out of scope: <what you are explicitly NOT doing>
<!-- rule: sizing-the-task > Step 3 -->

## [<YYYY-MM-DD HH:MM>] <task-id> — decision
<what was chosen + why, one line>

## [<YYYY-MM-DD HH:MM>] <task-id> — surprise
<what reality did vs. what the plan or brief assumed; which plan step was amended>   <!-- then replanning-on-surprise -->

## [<YYYY-MM-DD HH:MM>] <task-id> — evidence
<plan step N of M this proves>
ran: <command> → <verbatim result summary: counts, exit code>

## [<YYYY-MM-DD HH:MM>] <task-id> — escalation
Tier <old> → Tier <new>: <the trigger that fired>   <!-- rule: sizing-the-task > Re-sizing mid-task -->

## [<YYYY-MM-DD HH:MM>] <task-id> — blocked
<the specific question + who can answer it>

## [<YYYY-MM-DD HH:MM>] <task-id> — evidence (gate, T<n>)
ran: <test command> → <verbatim result summary: N passed, N failed, exit code>
ran: <lint or build command> → <verbatim result summary>
scope: <N files, all within plan or Done-when>; plus <undeclared-but-defensible edit> (declared)
checks: <brief acceptance check or Done-when check → the evidence that shows it, one per check>
<!-- rule: verifying-done > The gate procedure, step 5; per tier: verifying-done > What "done" means per tier -->

## [<YYYY-MM-DD HH:MM>] <task-id> — done
<what landed: merge SHA or PR URL>; evidence: <time of the gate evidence entry>

## [<YYYY-MM-DD HH:MM>] <task-id> — retro
<lessons added / merged / pruned (L<n>), or "no lessons — clean task">
