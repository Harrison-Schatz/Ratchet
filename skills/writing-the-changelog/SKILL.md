---
name: writing-the-changelog
description: Use whenever a change is about to land — invoked from landing-the-change before every PR/merge, at every tier including Tier 0 — and whenever the user says "add a changelog entry", "write the release notes", "what shipped in this version", or bumps a version. Also use the moment you find a project without a changelog, because it needs one. A changelog that reads like a commit dump tells a reader nothing they couldn't get from git.
---

# Writing the Changelog

The changelog is the one document written for someone who did not watch the work happen. It answers "what changed for me?", not "what did the author do?".

**Prevents:** failure mode #11 (documentation drift) in the release record specifically — a commit-dump changelog tells users nothing, and one that restates rationale kept elsewhere guarantees the two eventually disagree. Used strictly below: an **entry** is one release's whole block; a **bullet** is one line inside it.

**Dispatched as a sub-agent?** `landing-the-change` may run you concurrently with `syncing-the-docs`, `writing-the-issue`, and the retro. If so: do Steps 1–4 in the working tree, then stop before Step 5's land actions. **Your write-set is the changelog and the version file, nothing else** — not the issue records, not user-facing docs, not `.ratchet/`; those belong to the other three, and disjoint sets are the only reason the four can run at once. Do **not** commit, push, or write the worklog or PR body. Report back: the entry verbatim, the version you set and the consequence you mapped it from, any reference you were given and used, and any rule you had to bend and why. The dispatcher verifies your files before they ride into the commit — a "done" report is a claim, not evidence.

## Step 1 — Find the convention, or establish one

**Every project keeps a changelog.** It is not a project-type question and not the user's to opt out of: a release nobody can read is the drift this skill exists to prevent. So the first check is whether the project already has one.

Authority: the existing changelog, then the project's docs, then observable practice. Match an existing corpus's **layout** — heading level, group style, citation form. Do not match its voice or content where that breaks a Quality or Banned rule below; those override the corpus.

**No changelog yet → create one, in this change.** A single `CHANGELOG.md` at the repo root, in the shape below, holding this release as its first entry; say so in the PR. Don't ask permission and don't defer it to a follow-up task — a project acquires its changelog on the first release that lands after this skill is loaded, and backfilling the releases that predate it is separate, optional work. Other layouts are fine where already in place (one file per release under `changelog/<version>.md` trades discoverability for merge behavior — concurrent branches add separate files instead of racing on the same lines). Never convert an existing project as a side effect of adding an entry. **Every merge a release, or batched?** If batched, the bullet goes under `Unreleased` at the top now; the version and date are stamped when the release is cut, and that stamping is not a rewrite.

## Step 2 — Write the entry

```markdown
## 1.5.0 — 2026-08-10

### Added
- `--since <date>` limited a sync to entries changed on or after that date. (#41)

### Changed
- Large accounts finished exporting instead of timing out (the query now streams). (#38)

### Removed
- `--legacy-csv` was removed; `--format csv` produced the same file.

## 1.5.1 — 2026-08-12

Internal only — no user-facing change.
```

**Core rules**

- Write for **humans, describing user impact** — not a git commit dump.
- **Group by type: Added, Changed, Removed, Fixed**, in that order, omitting empty groups. Those four are the floor: a project already using Keep a Changelog's `Deprecated` and `Security` keeps them in its order (Added, Changed, Deprecated, Removed, Fixed, Security). Without those groups a deprecation notice is **Changed** and a security fix is **Fixed**. Headings or inline labels — match what's there.
- **Newest first**, each with its version and an **ISO 8601 date** — the release's date, not the day you wrote the bullet.
- **Every release gets an entry**, at every tier. No silent releases.
- **Link out when something else holds the depth** — a tracker ticket, an issue doc, or the PR. Match the project's citation form, or `(#N)` if it has none yet. A change with nothing deeper behind it gets no reference.

**Quality rules**

- **Impact first, mechanism in parentheses** — as in the Changed bullet above. Never the mechanism alone, never the reverse order.
- **One sentence per bullet**, 40 words or fewer, at most one parenthetical **besides the reference**. Longer means it carries two facts and wants to be two bullets.
- **Fewer words, more clarity.** Every line earns its place — if the same intent fits in fewer words, use fewer.
- **Past tense**, describing the release's effect rather than the author's action: "the export finished", never "fixed the export"; a removal reads "was removed". (Carve-out: a standing condition that is still true reads present — "requires a restart" — because past tense would imply it stopped being true.)

**Banned**

- Raw commit logs as entries.
- "Misc fixes and improvements", and every variant of it.
- **A bullet whose content is that something did *not* change** ("existing exports were unaffected"). An entry may still report that a whole release was internal-only — see `1.5.1` above: that is a statement about the release, not a change bullet.
- Contributor-facing sections: build notes, refactor commentary, "for maintainers".
- **Deploy and run instructions** — they belong in the PR/release description and the project's deploy doc. (What a *user* should use instead of something removed is impact, not operations; keep that.)

## Step 3 — One sentence, one reference

A bullet carries one sentence, plus a reference when something else holds the depth. That somewhere is whatever the project already uses as its durable record — a tracker, or in-repo issue docs where there is none (`writing-the-issue` decides which, and writes it). History, alternatives, known limitations and declined findings live there, never here. **Detail has exactly one home:** if the bullet and the record it cites explain the same thing, the bullet is wrong — cut it back to a sentence and the reference. Equally, don't mint a record just to give a bullet something to point at.

## Step 4 — Cross-check magnitude against the version

**The changelog explains content; the version explains magnitude.** Wherever the version can carry magnitude you need both, and they must not contradict each other. Read the current version from wherever the project keeps it — a version file, the latest tag, the package manifest.

**SemVer at 1.0 or above** — map by user-facing consequence, not by heading name: anything a user relied on breaks or disappears → MAJOR; a new capability, or any compatible change a user can observe (a changed default, new output ordering, a deprecation notice) → MINOR; everything else → PATCH. A mixed release takes the **largest** bump it earns, and a bug fix that breaks code written around the bug is MAJOR whatever heading it sits under. An **internal-only or docs-only release is PATCH** whichever types it carries — no interface changed shape.

**Pre-1.0** — breaking changes are expected here, so they do not force MAJOR: MINOR for anything breaking or added, PATCH otherwise. Never let 0.x reach 1.0 as a side effect; 1.0 is a stability declaration, not an arithmetic result.

**CalVer, build numbers, or no versioning** — the version carries no magnitude, so there is nothing to cross-check *and nothing warning the reader either*. The entry must carry that instead: say in the bullet that the change breaks something.

Batched releases: the cross-check runs when the release is cut, across the accumulated bullets — whoever stamps the version owns it. And it is **forward-only** — a shipped release whose number disagrees with its content stands as shipped; record the discrepancy in the durable record and move on, because renumbering breaks every reference that points at it.

## Step 5 — Land it, and leave history alone

The entry and any version bump are **part of the change** — same commit, authored by whoever made it. A changelog written afterwards is written from the commit log, which is the failure the Core rules exist to prevent. If the version is read at runtime or asserted by a test, re-run the suite after bumping; the gate never saw that file. **Shipped entries are history.** Correcting a factual error in place — a stale path, a wrong number — is `syncing-the-docs`' auto-fix lane and stays legal. Rewording, reordering or regenerating them while landing something else is not.

Reformatting a whole corpus to a new convention is legitimate work. The guard is the part that gets skipped, so it is specific:

- **Tier 2 minimum however few files it touches** — a brief and a review, never a self-sized patch. File count is the wrong instrument when the diff rewrites shipped history.
- **The user's go-ahead names the scope**, shown as a before/after on one real entry. "Tidy up the changelog" is not a go-ahead.
- **Snapshot the corpus before the first edit.** Byte-identity has nothing to compare against afterwards, and an agent that rewrites first can only verify against its own output.
- **Afterwards, show the user** that the set of release headings is unchanged and that every heading, date and reference is byte-identical.
- **Every fact a bullet loses must land somewhere** — a durable record, or a demonstration that it was false. Prose is the only part a reader reads, so "headings preserved" is evidence about anchors, not about content.

Missing any of these, it is not a reformat — it is data loss with a tidy diff.

## Stop conditions

- You cannot state a user-facing impact. Either it is internal-only — say so plainly at entry level — or you do not yet understand what you built.
- The honest bullet would disclose an unfixed vulnerability. Write the neutral sentence, put the detail in the durable record, and raise it with the user.
- Two concurrent branches claim the same version. Settle the number first; the entry follows it.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll paste the commit subjects and tidy them later" | Later is the release. Subjects describe authorship, bullets describe consequence. |
| "The detail matters, I'll keep it in the bullet too" | Then it lives twice and one copy rots. One sentence, one reference. |
| "Tiny internal change, it doesn't need an entry" | Every release gets one; "internal only, no user-facing change" is complete and useful. |
| "The old entries are inconsistent, I'll fix them while I'm here" | Tier 2, scoped go-ahead, snapshot first, and every dropped fact rehomed. Otherwise no. |
