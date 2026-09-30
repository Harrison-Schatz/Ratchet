<!-- Template for .ratchet/state/<task-id>.md. Format owned by `keeping-state` (state/<task-id>.md — the per-task snapshot).
     If this file and that skill disagree, the skill wins.
     Delete this comment when you copy. -->
# Task state: <task-id>
updated: <YYYY-MM-DD HH:MM>
owner: <person or agent name>
tier: <2 | 3>
phase: <sizing | briefing | planning | executing | verifying | landing | retro>
step: <N> of <M plan steps | - until the plan exists>
branch: <branch name | - until branched>
owned paths: <repo-relative glob>, <repo-relative glob>   <!-- rule: keeping-state > Ownership -->
worktree: <checkout path>                      <!-- optional: separate checkout -->
stack: <running-env label, host:port or url>   <!-- optional: dev environment -->
PR: <url>                                      <!-- optional: once opened -->
app: <clone folder owning the paths>           <!-- optional: multi-repo workspace -->

## Decisions on record   <!-- optional: decisions the next session must not re-argue -->
- <decision, one line>

## Next action
<one imperative sentence a stranger could execute>   <!-- rule: keeping-state > state/<task-id>.md — the per-task snapshot -->

## Blocked on
<nothing | the specific question + who can answer it>

## Pointers
brief:   .ratchet/briefs/<task-id>-brief.md
plan:    .ratchet/plans/<task-id>-plan.md
worklog: .ratchet/worklog/<task-id>.md
review:  .ratchet/review/<task-id>-*.md
