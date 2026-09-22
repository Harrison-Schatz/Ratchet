# Changelog

Newest first. Grouped by Added / Changed / Removed / Fixed; each release dated ISO 8601.

## Unreleased

### Added
- `.ratchet/issues/` joins the state layout as the default home for problems found while developing the application, with `issues/README.md` pointing at a project's own tracker or corpus when one exists (`keeping-state`, `writing-the-issue`, README). (#8)

### Changed
- `resuming-work`, `executing-with-checkpoints`, `replanning-on-surprise`, `planning-the-work`, and `landing-the-change` now name the per-task state file for "Next action", phase, and blocked-on checks (the roster has no such fields since the per-task split). (#7)
- `ponytail-review` folded its "When NOT to be lazy" section into the floor list, dropping language-specific test advice and hardware-specific examples that contradicted `testing-by-default`. (#7)
- `debugging-to-root-cause` Step 2 states the evidence constraint (fix at the origin) instead of a technique catalog. (#7)
- `writing-the-changelog`, `resuming-work`, and `retrospecting` express length as outcome rather than word or sentence counts. (#7)
- `writing-the-issue` chooses between two homes — the project's existing tracker or corpus, else `.ratchet/issues/` — and no longer treats a worklog line as a record. (#8)
- `characterizing-legacy-code` Step 3 asks for captured output instead of the assert-a-wrong-value ritual, matching the child methodology's `characterizing-the-behavior`. (#8)
- `keeping-state` gains a backward-compatibility note: an older `.ratchet/` without `issues/`, or a project with an existing corpus, gets a pointer README on the first record; nothing moves. (#8)
- `delegating-to-agents` briefs the coexistence clauses (scratch dir, which checkout, one suite runner per shared environment, no unreproduced reds) and sweeps for stray files before the checkpoint. (#9)
- `reviewing-the-diff` briefs both lenses on a seam that has no caller by design, and reads both records in full before applying either. (#9)
- `verifying-done` keeps a layout or styling claim at "expected, pending visual" until a person or a browser observed the build, and adds two Hard rules: prose is a claim too, and a proof gate must be able to fail. (#9)
- `landing-the-change` treats landing on a moving base as a loop ending on a green run for the latest commit, and re-reads the project's landing contracts as of the current base before writing the entry. (#9)
- `fresh-eyes-review` Checklist 2 (Quality) gains a Deletions check: the inputs that fed removed code — a parameter, a config key, a payload field — are named as orphans or as deliberate. (#9)
- `testing-by-default` adds the declared-populated-proof row: a schema or invariant change is run against a realistic snapshot, and the record says what it could have failed on. (#9)
- README's skill-shape paragraph now says a step edit re-reads the `description` and Rationalization table for a surviving contradiction. (#9)

### Removed
- Migration-relative and authoring-note phrasing in `keeping-state`, `landing-the-change`, and `syncing-the-docs`. (#7)
