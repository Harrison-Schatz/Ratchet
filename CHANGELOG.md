# Changelog

Newest first. Grouped by Added / Changed / Removed / Fixed; each release dated ISO 8601.

## Unreleased

### Changed
- `resuming-work`, `executing-with-checkpoints`, `replanning-on-surprise`, `planning-the-work`, and `landing-the-change` now name the per-task state file for "Next action", phase, and blocked-on checks (the roster has no such fields since the per-task split). (#7)
- `ponytail-review` folded its "When NOT to be lazy" section into the floor list, dropping language-specific test advice and hardware-specific examples that contradicted `testing-by-default`. (#7)
- `debugging-to-root-cause` Step 2 states the evidence constraint (fix at the origin) instead of a technique catalog. (#7)
- `writing-the-changelog`, `resuming-work`, and `retrospecting` express length as outcome rather than word or sentence counts. (#7)

### Removed
- Migration-relative and authoring-note phrasing in `keeping-state`, `landing-the-change`, and `syncing-the-docs`. (#7)
