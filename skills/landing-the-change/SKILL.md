---
name: landing-the-change
description: Use when the verifying-done gate has passed and the work needs to become permanent — committing final state, merging, opening a PR, or handing off. Also use when the user says "ship it", "merge it", "open a PR", or asks what happens to the branch. Closes the loop: land, record, reset state, trigger the retro.
---

# Landing the Change

Verified work isn't finished until it's landed where the project expects it, the record says so, and the state file is clean for whoever comes next. This is mechanics, done carefully — the thinking already happened at the gate.

**Prevents:** failure mode #3's tail (work declared done but never actually integrated) and #5 (state left dirty, poisoning the next resume).

## Step 0 — Preconditions

- `verifying-done` passed at this tier, evidence entry in the worklog. Not the case → STOP, go there. This skill never re-argues the gate; it also never proceeds without it.
- Tier 2+: `reviewing-the-diff` resolved (Critical/Important closed).

## Step 1 — Determine the landing convention

In order of authority:
1. The user's explicit instruction for this task.
2. `.ratchet/LESSONS.md` / project docs (CONTRIBUTING, CLAUDE.md) stating the convention (PRs always? merge to develop? squash?).
3. Observable practice: `git log` shows merge commits vs. squashes; a remote with PRs implies PRs.
4. None of the above → ask ONE question: "Verified and ready: <task>. Merge to <default>, or open a PR?" (Tier 0 may land directly on the default branch per its sizing; everything else came from a branch.)

## Step 2 — Land it

**Tier 2/3 — run the docs audit and the retro in parallel.** Both are slow and independent, so don't do them end-to-end. When both have real work (the diff touched public surface docs describe, AND the worklog holds surprises/escalations/review findings worth mining):

- **Dispatch `syncing-the-docs` as a sub-agent** (per `delegating-to-agents`) — it's diff-driven and auditable, the same profile as the review agents in `reviewing-the-diff`. Brief it with `BASE..HEAD`, the public-surface delta, and the doc set; forbid it from touching `.ratchet/` (that's the retro's write-set — the ban keeps the two disjoint, which is what makes the parallelism safe). It edits the working tree and reports its edits back; it does NOT commit or push.
- **Run `retrospecting` yourself, in the main session — never as a sub-agent.** It needs the friction you actually lived (not just what the worklog caught), and it writes `LESSONS.md`, which every session loads; that judgment isn't delegable. Do its mining and lesson-drafting now, while the docs agent runs; finalize it in Step 4 once the land is known-good.

When the sub-agent returns, **verify its doc edits like the gate** (`delegating-to-agents` Step 3 — read the diff; real and in-scope?) before they ride into the commit; a "done" report is a claim, not evidence. Docs land in the SAME commit as the code — a docs "fast follow" is a fast never. Only one of the two has real work → skip the fan-out and run it inline. (Tier 0/1: skip the docs audit unless the change touched public surface that docs describe.)

**Common path (PR):**
```
git push -u origin <task-id>
gh pr create --title "<imperative summary>" --body "<goal + acceptance checks demonstrated + evidence summary + anything declared out of scope>"
```
The PR body is mostly written already — lift it from the brief and the gate's evidence entry. Tier 2+: link the brief/plan paths in the body.

**Merge-locally path:** merge (or squash-merge per convention) into the base branch, **re-run the test suite on the merged result** (the merge itself is a new state the gate never saw — one command, real protection), push if a remote exists, delete the task branch.

**Hand-off path** (user will handle integration): push the branch, report its name and the evidence summary, leave the branch alone.

Whatever lands: never force-push shared branches; never delete work without the user's explicit confirmation (destructive ops get a typed go-ahead).

## Step 3 — Close the record

1. Worklog `done` entry (in the task's `.ratchet/worklog/<task-id>.md`): task id, what landed (merge SHA / PR URL), evidence pointer.
2. Remove this task's row from the roster (STATE.md) — move it to "Recently landed" if useful — and archive/delete its `.ratchet/state/<task-id>.md`. The task's `.ratchet/worklog/<task-id>.md` STAYS in place as the permanent record (the journal is not deleted; only the ephemeral state snapshot is). Do NOT flip a global idle while other tasks are active; only THIS task closes. (Tier 3 with milestones remaining: keep the row, point its "Next action" at the next milestone.)
3. Uncommitted strays in the tree that aren't yours → leave them, mention them; yours → they should not exist (the scope check would have caught them — if one appears now, resolve it before closing).

## Step 4 — Finalize the retro

- **Tier 2/3:** `retrospecting` already started in Step 2, concurrent with the docs sub-agent (when it had lessons to mine). Finalize it here, now that the land is known-good — fold in any surprise the land itself produced (a CI fail or a conflict is retro material too), then write its worklog `retro` entry. Not optional. (Nothing to mine back in Step 2 → run it here from scratch.)
- **Tier 0/1:** invoke `retrospecting` only if something surprised you (any `surprise` entry in this task's worklog, or a lesson candidate you noticed). Zero surprises → skip it by rule; the task is closed.

## Stop conditions

- Merge conflicts with the base branch → resolve only if trivially mechanical; anything semantic re-runs the suite and, if behavior questions arise, goes back through `verifying-done` on the resolved result.
- CI fails on the PR after local gates passed → that's a `surprise` (environment differs from local); `debugging-to-root-cause`, and the lesson about the difference likely belongs in LESSONS.md.

## Rationalization check

| Thought | Reality |
|---|---|
| "Gate passed locally, skip re-testing the merge" | The merged tree is code nobody has run. One suite run; cheap; occasionally a lifesaver. |
| "I'll clean up STATE.md next session" | Next session might be a different agent reading a state file that lies. Thirty seconds, now. |
| "Retro later, while it's fresh → actually never" | Correct, that's why Step 4 fires it now, before this skill reports completion to the user. |
| "Docs then retro, one after the other — simpler" | They're independent and both slow; sequential wastes the overlap. Delegate the docs audit, run the retro yourself, concurrently. |
