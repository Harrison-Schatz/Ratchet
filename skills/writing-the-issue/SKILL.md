---
name: writing-the-issue
description: Use the moment a problem is identified and not being fixed now — work deferred out of scope, a review finding declined, a known limitation accepted, a bug found while doing something else — and at every land, to flip the status of records the release resolved. Also use when the user says "log that", "write it up", "note it for later", or asks where a past decision was recorded. Dispatchable as a sub-agent from landing-the-change. A deferral nobody can find is a deferral that gets re-discovered, re-argued, and deferred again.
---

# Writing the Issue

A problem you are not fixing right now needs a home that outlives the session that found it. This skill decides which home, and what goes in it.

**Prevents:** failure mode #5 (context loss between sessions) in its durable form — not "what was I doing", but "why is this like this, and did someone already decide not to fix it?". It also makes `writing-the-changelog` possible: a release entry is one sentence plus a reference, and that only works if the reference exists.

## Step 1 — Which home

A record's job is to be findable by someone who never had your session. Choose in this order:

1. **The project's issue tracker**, if it has one. Never build a parallel corpus beside GitHub Issues, Jira or Linear — cite the ticket and stop. (Exception: a tracker the repo's own readers cannot open, because it is private to one team or behind a login they lack, fails the legibility test that makes option 3 exist. Then it is not a home.)
2. **The worklog** (`keeping-state`) — durable *only* if it is version-controlled with the repo. One the user chose to gitignore, or that lives outside the repo, is invisible to a repo reader.
3. **In-repo issue docs**, when the problem must be legible to a repo reader and neither of the above holds.

Only option 3 needs the rest of this skill. Options 1 and 2 are a link and a line.

## Step 2 — Write the record

Sequentially numbered from `#1` in `issues/`, unless the project already has somewhere better:

```markdown
# #12 — Scheduled jobs silently skip the hour a clock change removes

Status: open · opened 2026-08-10

## Problem            <what is wrong, present tense, verified against current code>
## History            <what was tried or deliberately deferred, dated — on a new record, the one line saying when it was identified>
## Proposed behavior  <what "fixed" would mean, not a patch>
## Why it matters     <who is affected and how badly; "nobody yet, but" is legitimate>
```

**Verify the problem against current code before writing it.** A record describing behavior that no longer exists is worse than no record: it will be read as live, and someone will spend an afternoon on it. If you cannot reproduce or locate it, say so in `Problem` — an unconfirmed report is a legitimate record as long as it admits what it is.

`opened` is the day the file was written. Add `(raised <date>)` when the problem was found earlier, so a backfilled record does not read as new. Status is **`open`**, **`closed (<date>)`** once a change resolved it, or **`declined (<date>)`** when it was judged not worth fixing — that last one earns its keep by stopping the same finding being re-raised at every review. Extending the vocabulary is fine; leaving it to each writer is not, so the first record in a fresh corpus pins it in the folder's README.

Numbering collides when two branches claim `#N` at once. That collides as an add/add conflict, which is the point — but re-check the highest number at land, and when a dispatcher assigns you a number, use the one you were given rather than picking your own.

## Step 3 — Keep it true

- **Records are living documents.** Rewrite one freely as understanding improves. (Changelog entries are the opposite — `writing-the-changelog`.)
- **A change that resolves a record flips its status in the same commit.** Skip that and the corpus silently becomes a list of things that look open forever, which is indistinguishable from an abandoned one.
- **Detail lives here, not in two places.** Whatever cites this record — a changelog bullet, a roster line, a code comment — carries one sentence and the reference. If the citation explains as much as the record, one of them is wrong.
- **Declining is a decision, not a dismissal.** A `declined` record says what was judged and why, so the next reviewer can disagree with the reasoning instead of re-deriving it.

## Dispatched as a sub-agent?

`landing-the-change` may run you concurrently with `syncing-the-docs`, `writing-the-changelog`, and the retro. If so:

- **Your write-set is the records and nothing else.** Not the changelog, not the version file, not user-facing docs, not `.ratchet/`. Those belong to the other three, and the only reason the four can run at once is that the sets are disjoint — one stray edit can silently clobber another agent's exact-match `Edit`.
- **Do not commit, push, or write the worklog or PR body.** The dispatcher lands your files in the same commit as the code.
- **Use the number you were assigned.** Do not scan for the next free one; a concurrent land may hold it.
- **Report back:** each record written or status flipped, one line each, with its number; anything you could not verify against current code; and anything you declined to record, with the reason. The dispatcher verifies your files before they ride into the commit — a "done" report is a claim, not evidence.

## Stop conditions

- The problem turns out to be live and cheap to fix. Then fix it; a record is for what you are *not* doing.
- You cannot say what "fixed" would look like. Write `Problem` and `Why it matters`, and say plainly in `Proposed behavior` that the shape of a fix is unknown — an honest gap beats an invented design.
- The record would have to describe an unfixed vulnerability in a repo that is public or widely readable. Write the neutral version and raise the detail with the user directly.

## Rationalization check

| Thought | Reality |
|---|---|
| "I'll remember to bring this up" | You are one session from not existing. Write it down. |
| "It's minor, a worklog line is enough" | Only if the worklog ships with the repo. Otherwise the next reader never sees it. |
| "The reviewer was wrong, nothing to record" | Then record *that*, as `declined` with the reasoning — or defend it again next review, from scratch. |
| "I'll write it properly once the fix is scoped" | The record is what makes scoping possible later. Unwritten problems don't get scoped, they get rediscovered. |
