<!-- Template for .ratchet/plans/<task-id>-plan.md. Format owned by `planning-the-work` (Step 2 — Write the plan).
     If this file and that skill disagree, the skill wins.
     Delete this comment when you copy. -->
# Plan: <title>                    <!-- task-id matches the brief -->
Brief: .ratchet/briefs/<task-id>-brief.md   <!-- optional: "(approved <YYYY-MM-DD> by <who>)" — the brief's approval; rule: planning-the-work > Step 4 -->

## Territory notes   <!-- optional: the Step 1 map, kept; rule: planning-the-work > Step 1 -->
- <file or fact checked before writing steps>

## Steps
### Step 1: <outcome, not activity — the state that exists when the step is done>
- Files: `<path>` (<create | modify>)
- Approach: <2-4 sentences; sketch interfaces or signatures a stranger would need, never full implementations>
- Test discipline: <test-first | characterize-first | spike-then-stabilize>   <!-- rule: testing-by-default -->
- Prove it: <executable command and the observable result that counts as pass>
- Checkpoint: commit "<message>"

### Step 2: <outcome, not activity>   <!-- same five bullets, one block per step -->

## Step ordering
<one line: why this order; which steps could run in parallel safely (for delegating-to-agents)>

## Change log
<empty at birth. Every amendment appends: date, what changed, why, which steps were invalidated/added. Maintained by replanning-on-surprise.>
