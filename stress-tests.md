# Ratchet — Stress-Test Traces

Five scenarios traced skill-by-skill against the catalog as written. Where a trace exposed ambiguity, the skill was fixed (fixes listed at the end), and the traces below reflect the post-fix text.

---

## Scenario 1 — "Fix this typo in the README."

| # | Skill | What happens |
|---|---|---|
| 1 | `using-ratchet` | Step 0: no `.ratchet/STATE.md` (or idle). No LESSONS.md. Routes "change request" → `sizing-the-task`, before any clarifying question. |
| 2 | `sizing-the-task` | Step 1 risk surfaces: none. Step 2: 1 file, no interface change, acceptance check statable in one sentence ("README line 12 reads 'the', not 'teh'"), one revert undoes it → **Tier 0**. No record written before the work. Hand-off: "implement now." |
| 3 | — | Edit the file. Tier 0 may work directly on the default branch (using-ratchet rule 3 binds only "past Tier 0"). |
| 4 | `verifying-done` | T0 row: run the proving observation fresh (`grep -n "teh" README.md` → no hits; the corrected line shown). Repo has no `.ratchet/` → rule says do NOT create one for Tier 0: evidence goes in the commit message + report. Scope check: diff touches 1 file, matches request. |
| 5 | `landing-the-change` | Convention: direct commit to default branch is legitimate for Tier 0. Commit `docs: fix typo in README ("teh" → "the"; grep verified clean)`. Step 4: retro only if a surprise occurred — none → skipped **by rule**. |

**Artifacts on disk afterward:** one commit. Nothing else — no `.ratchet/`, no brief, no plan, zero questions asked.
**Overhead vs. raw edit:** the sizing walk (~30 seconds) and one fresh grep. This is the proportionality requirement holding.

---

## Scenario 2 — "Add OAuth login to our existing Express app" (no tests exist)

| # | Skill | What happens |
|---|---|---|
| 1 | `using-ratchet` | Orient (no state in flight) → route to `sizing-the-task`. |
| 2 | `sizing-the-task` | Step 1: **auth = risk surface → minimum Tier 2** regardless of size. Worklog `sizing` entry written; `.ratchet/` created (first Tier 1+ task); STATE.md → phase `briefing`. |
| 3 | `writing-the-brief` | Explore codebase first (Express version, session handling, existing user model). Questions one at a time: which provider(s)? link to existing accounts or replace password auth? session vs JWT? Approaches (e.g., passport vs hand-rolled vs Auth0) with recommendation. Brief saved to `.ratchet/briefs/…` with observable acceptance checks ("GET /login redirects to Google consent; callback with valid code yields session cookie; invalid state param → 403, test proves it"), out-of-scope ("no account-merge UI, no refresh-token rotation"), risk note: "auth → second independent reviewer pass at review." **Blocking user approval.** STATE → `planning`. |
| 4 | `planning-the-work` | Step 1 territory map finds: touched code has no tests AND **no test infrastructure exists** → plan Step 1 = bootstrap minimal runner (jest/vitest + one smoke test + `npm test` wired). Subsequent steps touching existing code (app bootstrap, session middleware) are marked `characterize-first`; new OAuth modules marked `test-first`. Each step: files + proof command + checkpoint. STATE → `executing`. |
| 5 | `executing-with-checkpoints` | Setup: branch created; baseline = `no test infrastructure` recorded; gate criterion becomes "the suite that exists by the end passes." Step 1 bootstraps the runner (proof: smoke test passes). |
| 6 | `characterizing-legacy-code` | Before touching the existing app bootstrap/middleware: blast radius = routes touched by session changes. Step 2 harness attempt fails — `app.js` connects to the DB at import time → STOP per Step 2.3. |
| 7 | `finding-seams` | Blocker named: "can't test routes because app construction has import-time side effects." Cheapest fit: **constructor seam** — extract `createApp(deps)` with a default preserving behavior; seam commit labeled `refactor: … (no behavior change)`. Return to characterizing. |
| 8 | `characterizing-legacy-code` | Pin current behavior through the seam via supertest: existing routes' status codes, session cookie behavior, the error path ("unknown route → current 404 body"). Tests green against unchanged code; committed separately. |
| 9 | `testing-by-default` / loop | OAuth routes and token handling built test-first (happy path, failure paths: bad state param, expired code, provider 500). Each step: prove → commit → evidence line → STATE tick. |
| 10 | `reviewing-the-diff` | Motion A, both checklists; **risk surface → second independent reviewer pass over the auth files** (different subagent). Findings triaged; Critical/Important fixed and re-checked. |
| 11 | `verifying-done` | T2: full suite + lint, every acceptance check in the brief individually demonstrated (curl transcript for the redirect chain), diff-vs-scope check against the out-of-scope list. Evidence entry written. |
| 12 | `landing-the-change` | Convention detected (repo has PRs) → push + PR; body lifted from brief + evidence. Worklog `done`; STATE → idle. |
| 13 | `retrospecting` | Mandatory at Tier 2. Mined from the record: "app has import-time side effects; always test through `createApp()`" → LESSONS.md L1 with `when/rule/because`. |

**Artifacts afterward:** `.ratchet/{STATE.md(idle), WORKLOG.md, LESSONS.md, briefs/…, plans/…}`, test infrastructure, characterization + feature tests, seam commit, feature commits, PR.

---

## Scenario 3 — "Build a CLI tool that syncs two APIs" (greenfield, ambiguous spec)

| # | Skill | What happens |
|---|---|---|
| 1 | `using-ratchet` → `sizing-the-task` | No risk surface yet (unless the APIs are payment/auth systems — Step 1 would catch that). New components + genuine ambiguity → **Tier 2**. (If discovery shows multi-session scope → Tier 3 and milestone decomposition in the brief.) `.ratchet/` created; STATE → `briefing`. |
| 2 | `writing-the-brief` | The ambiguity is the work: one-way or two-way sync? conflict policy (newest-wins? source-of-truth?)? full or incremental? auth/secrets handling (→ would add a risk surface)? failure semantics (partial sync = abort or resume?)? Approaches (e.g., timestamp diff vs changelog cursor) with recommendation. Acceptance checks made observable ("`sync --dry-run` prints a correct plan against fixture APIs; a record updated in A appears in B within one run; conflict per policy X, test proves it"). **User approval gates.** |
| 3 | `planning-the-work` | Greenfield: no characterization steps; all steps `test-first` (fixture/fake API servers are part of step 1's proof harness). Steps sized to provable outcomes; parallel-safe steps marked (e.g., the two API clients). |
| 4 | `executing-with-checkpoints` | Branch; baseline trivially green; loop with per-step commit + evidence + STATE tick. Optionally `delegating-to-agents` for the two independent API clients (disjoint files) — dispatcher re-runs each proof itself before checkpointing. |
| 5 | `reviewing-the-diff` → `verifying-done` → `landing-the-change` → `retrospecting` | As in scenario 2, minus the second reviewer pass (no risk surface). |

**Artifacts afterward:** state dir with brief/plan/worklog, fixture servers, tested CLI, branch/PR per convention, retro entry (possibly zero lessons).

---

## Scenario 4 — Mid-task disaster: tests at step 7 of 10 prove the chosen approach wrong

Context: scenario 3 in flight; step 7's proof fails — the incremental-cursor approach can't work because API B's changelog endpoint paginates inconsistently.

| # | Skill | What happens |
|---|---|---|
| 1 | `executing-with-checkpoints` | Loop step 3: proof fails. Not a typo-level bug — evidence contradicts the plan's *approach* → route to `replanning-on-surprise` (not "fix harder"). If the cause were unclear, `debugging-to-root-cause` first establishes WHAT is true (reproduces the pagination inconsistency, hypothesis written) — its three-strikes rule independently forces the same architectural stop if someone tries to patch through it. |
| 2 | `replanning-on-surprise` | Step 1: `surprise` worklog entry — expected (stable cursor) vs found (pagination breaks ordering; evidence: response transcripts). Step 2 classify: steps 7–10 assumed the cursor → **Structural**. (If NO approach could meet the brief's "incremental sync" acceptance check, it's **Intent-level** → user decides; trace continues on the structural branch.) |
| 3 | `replanning-on-surprise` Step 3 | Steps 7–10 struck through in the plan (kept, marked invalidated). Salvage audit: steps 1–6 (clients, models, dry-run) survive — they're committed, verified clicks. Replacement steps written for timestamp-window sync (files + proofs, planning-the-work rules + self-check on remainder). Change log entry: old → new → why. STATE.md step counts and Next action updated. |
| 4 | `executing-with-checkpoints` | Resumes at new step 7. **Escalation watch:** this was surprise #1; a second surprise on this task auto-triggers `sizing-the-task` re-size (the escape hatch — e.g., "this is really Tier 3; decompose"). |

**Artifacts afterward:** worklog surprise entry, amended plan with visible dead steps + change log, salvaged commits intact, STATE consistent. No silent fork: a fresh reader of the plan sees exactly what changed and why.

---

## Scenario 5 — Session dies halfway through scenario 3; a fresh agent must resume

Context: the session crashed during step 4 of the plan — after editing files but before the checkpoint.

| # | Skill | What happens |
|---|---|---|
| 1 | `using-ratchet` | Step 0: `.ratchet/STATE.md` exists, `status: active`, phase `executing`, `step 3 of 9` recorded as done, Next action = "Run step 4 (conflict resolver)". → `resuming-work` immediately; not treated as a new task. |
| 2 | `resuming-work` Step 1 | Reads STATE.md, LESSONS.md, the brief, the plan **including change log**, and the task's worklog entries (decisions: "newest-wins chosen 11:02"; surprises: none yet). |
| 3 | Step 2 verify | `git status`: uncommitted changes in `src/resolve.ts` — work EXISTS beyond the last recorded step. `git log`: commits match checkpoints 1–3. Test suite: green (matches last evidence entry). Diagnosis: died mid-step-4 after working, before recording. |
| 4 | Step 2 table row 2 | Read the diff. The partial resolver is half-written and its test doesn't exist yet → not verifiable → **revert to the last checkpoint** (stash the partial for reference is acceptable; trusting it isn't). Plan step 4 will re-derive it test-first. |
| 5 | Step 3 reconcile | STATE.md corrected (still step 3 done, next = step 4); worklog `decision` entry: "resumed; reverted unverifiable partial step-4 edits (stash ref)". Announce to user in 3 sentences: where things stand, what was reconciled, what happens next. No re-litigating the newest-wins decision — it's in the record. |
| 6 | `executing-with-checkpoints` | Re-enters the loop at step 4. The ratchet held: cost of the crash = one step, not the project. |

**Artifacts afterward:** corrected STATE.md, resume decision in the worklog, everything else as the dead session left it at the last click.

---

## Ambiguities the traces exposed → fixes applied

1. **Tier 0 in a repo with no `.ratchet/`** (Scenario 1). `verifying-done` demanded a worklog evidence line; creating a state directory for a typo violates proportionality. **Fix:** `verifying-done` T0 row + `keeping-state` now state: Tier 0 never creates `.ratchet/`; evidence goes in the commit message and the user report; the directory is born at the first Tier 1+ task.
2. **No test infrastructure at all** (Scenario 2). `executing-with-checkpoints` said "run the suite" against a project with no runner; `planning-the-work` assumed a discipline could run. **Fix:** `planning-the-work` Step 1 now mandates a bootstrap-runner first step when no infrastructure exists; `executing-with-checkpoints` setup gained a third baseline branch (`no suite exists` → record it; gate criterion becomes "the suite that exists by the end passes").

Checked and confirmed unambiguous without fixes: the Tier 0 default-branch exception (stated in both `using-ratchet` rule 3 and `landing-the-change`), the structural-vs-intent fork in `replanning-on-surprise` (classification table covers the disaster scenario both ways), the mid-step-death recovery (the `resuming-work` reconciliation table's second row decides it), and the second-surprise escalation trigger (stated identically in `sizing-the-task` and `replanning-on-surprise`).
