---
name: syncing-the-docs
description: Use when landing any Tier 2/3 change (invoked from landing-the-change before the PR/merge), when the user says "update the docs", "sync documentation", or "post-ship docs", or whenever a diff adds, renames, or removes public surface — commands, flags, config options, endpoints, env vars — that README, ARCHITECTURE, CONTRIBUTING, CHANGELOG, or CLAUDE.md might describe. Docs that contradict shipped code are misinformation with a project logo.
---

# Syncing the Docs

Documentation is the human-facing half of "disk beats conversation" — it's what future readers (and future agent sessions) trust instead of re-deriving the truth. This skill audits every doc against the diff that's about to land, fixes what is *factually* wrong, asks about what is *narratively* wrong, and flags what is missing — without ever generating filler.

**Prevents:** failure mode #11 (documentation drift — docs contradicting shipped code mislead users and future sessions; stale docs are worse than no docs because they're trusted).

## Step 1 — Inventory the change and the docs

1. `git diff <base>...HEAD --stat` and `--name-only` — what shipped.
2. Find the docs: `*.md` files at depth ≤2 (skip `node_modules`, `.git`, `.ratchet/` — the worklog is not user documentation).
3. Extract the **public-surface delta** from the diff: new/renamed/removed commands, CLI flags, config options, API endpoints, env vars, user-visible capabilities. This list drives everything below.

## Step 2 — Coverage map (audit lens, never a generator)

For each public-surface item, check coverage in four kinds: **reference** (what it is — tables, API docs), **how-to** (task-oriented example), **tutorial** (newcomer walkthrough), **explanation** (why it's designed this way).

```
/sync --dry-run    reference:✅ README   how-to:❌   tutorial:❌   explanation:❌
FooProcessor       reference:❌          how-to:❌   tutorial:❌   explanation:❌
```

Zero-coverage items are **critical gaps**; reference-only items are **common gaps**. Gaps get *flagged* (Step 5), never auto-filled — writing a missing guide is a new task that goes through `sizing-the-task`, not a side effect of landing.

## Step 3 — Per-file audit with the factual/narrative split

Read each doc fully, cross-reference against the diff, and classify every needed edit:

**Auto-fix (factual, the diff is the evidence):** stale paths, counts, version-number mismatches, adding a row to an existing table/list, fixing cross-references to renamed things, project-structure trees, commands that no longer exist as written. Apply with Edit; one summary line per file stating *specifically* what changed ("README: added `--dry-run` to flags table, fixed `sync.yml` path").

**Ask first (narrative, intent territory):** positioning/philosophy text, security-model descriptions, removing any section, rewrites longer than ~10 lines, anything where the "fix" requires deciding what the project *means* rather than what the code *does*.

**Never:**
- Regenerate or reorder CHANGELOG entries. Polish wording in place with exact-match Edit only; never Write over the file. The entry was derived from the actual change — it is history, and history doesn't get rewritten by a docs pass.
- Bump VERSION silently. If a VERSION file exists and wasn't bumped, ask once; default recommendation is no bump for docs-only edits.
- Auto-edit architecture diagrams (ASCII/Mermaid). Diagram drift — entities renamed/removed in code but still in the picture — is flagged in Step 5; updating a diagram correctly needs human judgment.

## Step 4 — Cross-doc consistency and discoverability

1. Do README, CLAUDE.md, and CONTRIBUTING agree about features, commands, and structure? Fix factual contradictions (version mismatch); ask about narrative ones.
2. Is every doc reachable from README or CLAUDE.md? An unlinked doc is invisible; add the link or flag it.

## Step 5 — Record and land

1. Doc edits join the SAME change — committed on the task branch so docs and code land together, not in a follow-up that never comes.
2. Append the worklog `evidence` entry: files updated (one line each), gaps flagged, diagram drift noted.
3. If a PR exists, add a `## Documentation` section to its body: what changed per file, plus a `### Documentation debt` list of critical/common gaps and drifted diagrams. Debt is declared, not hidden.
4. Return to `landing-the-change` where it left off.

Run standalone (user asked outside a landing)? Then it IS a task: size it first (`sizing-the-task` — usually Tier 0/1), and the gate evidence is Step 5's summary.

## Stop conditions

- The diff changes behavior that docs describe, but you cannot tell what the new truth is → that's a question for the user, not a guess in prose.
- The audit reveals the docs and the brief disagree about what was built → `replanning-on-surprise` (intent-level), not a docs edit.

## Rationalization check

| Thought | Reality |
|---|---|
| "Docs can be a fast follow" | Fast follows are slow nevers. Same branch, same PR, same gate. |
| "I'll just regenerate the README section, it's easier" | Regeneration deletes facts you didn't know were there. Audit and edit; generate only when a human asked for new docs. |
| "The gap is obvious, I'll write the missing guide now" | Unsized writing is scope creep with good intentions. Flag it; let it be sized. |
| "Nobody reads CONTRIBUTING" | The next agent session does — and trusts it. |
