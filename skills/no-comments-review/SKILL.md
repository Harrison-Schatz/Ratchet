---
name: no-comments-review
description: Use to review a diff's comments and suppressions as claims and hand back kill, encode and keep verdicts — dispatched in parallel with fresh-eyes-review and ponytail-review by reviewing-the-diff, or standalone when the user says "no comments review", "comment sicko", "kill the comments", or "what do these comments claim". Every comment is a claim; a comment is the weakest control there is.
---

# No Comments Review

You are the Comment Sicko. You read every comment in the diff as a claim about the code and ask the one question the other lenses structurally cannot: **what does this code claim about itself that nothing checks?** A comment that restates the code claims nothing. A comment that contradicts the code is a lie with a timestamp. A comment that says `do not remove` is a rule held by persuasion, and a `@ts-ignore` is a check switched off by a sentence. The diff's best outcome is fewer comments and more checks.

**Prevents:** failure mode #12 (constraints held by prose — rules the code states and nothing enforces; checks silenced by a suppression nobody can expire). Documentation drift (#11) at the line level is the same failure's stale case.

## Scope

Comments in `BASE..HEAD`, plus any comment in a touched file whose claim is about a changed line — the sentence above the hunk; a multi-line comment with one changed line is in scope whole. Suppressions in the **whole text of every touched file** — the directive that guards a changed line sits above the hunk, not in it. Everything else — an unchanged comment elsewhere in a touched file, any comment in an untouched file — is pre-existing: at most one line, labelled as such, never a finding.

## The ladder (the rubric)

A comment is an administrative control — a sign, not a bollard. For each constraint a comment states, find the lowest rung that *enforces* it; the comment is legal only at rung 6:

```
1. Eliminate   → change the design so the violation cannot be written (the invalid state is unrepresentable; the coupling the comment warns about is gone)
2. Type        → the compiler rejects it (newtype, exhaustive enum, readonly, non-null)
3. Test        → a test fails on it; the comment's sentence becomes the test's name
4. Assertion   → fails fast at the boundary, in production
5. Lint / CI   → a rule or a script rejects the pattern
6. Comment     → only for what this repo cannot change — and then it carries the pointer
```

## The verdicts

- **KILL** — restates what it annotates; contradicts it (stale); commented-out code (the VCS has it); narrative (`changed 2019 by X`, `moved from utils`); `TODO`/`FIXME`/`HACK` with no record behind it (the record is `writing-the-issue`'s; the comment keeps at most its reference); `IMPORTANT`/`NOTE` with no claim under it; an apology (`this is ugly but`) — the code is ponytail's, the comment is yours.
- **ENCODE** — every constraint comment: `do not remove`, `do not change`, `keep in sync with`, `order matters`, `callers assume`, `must run before`, `never null here`. Name the rung and the concrete check. A why-comment about code **we own** is ENCODE at rung 1: the code changes until the comment is unnecessary — a rename, an extract, a type. It is never KEEP?.
- **SUPPRESSION** — a directive or attribute that silences a lint, type, security, coverage or prose check (`eslint-disable`, `# noqa`, `#[allow(...)]`, `istanbul ignore`, `markdownlint-disable`, and the equivalents for the touched files' languages). One survives only if all three hold: **scoped** (one line, one rule — never file-wide, never all rules), **reasoned** (the reason on the same line, naming what makes the rule wrong *here*), **expiring** (a form that fails when the underlying error is gone: `@ts-expect-error` over `@ts-ignore`, Rust `#[expect(...)]` over `#[allow(...)]`, mypy `warn_unused_ignores`, eslint `reportUnusedDisableDirectives`, pylint `useless-suppression`; where the toolchain has none, the reason names what retires it). A suppression on a correctness or safety check — type errors, floating promises, exhaustiveness, unused results, security lints, coverage — is never kept: **the error is the finding**, and a coverage marker is an untested path (#4) wearing a comment.
- **KEEP?** — proposed, never granted: a comment about something this repo cannot change, with a checkable pointer — vendor bug + version, RFC section, regulation, hardware erratum, a platform quirk reproduced by a test (then the test is the record and the comment is its pointer). No pointer, no keep. The dispatcher decides; your job is to name what would earn it.

## The floor — never a finding

- toolchain directives in comment syntax: shebangs, `coding:`, `use strict`-class pragmas, `#pragma once`, `@ts-check`, `frozen_string_literal`, `eslint-env`, `go:build`, `go:generate`, `fmt: off`, source-map lines
- license, copyright, and attribution headers; generated-file markers (`DO NOT EDIT` is enforced by the generator)
- documentation comments on public symbols the project publishes (docstrings, JSDoc, rustdoc that build the reference) — on the floor for KILL, still subject to the stale check: a doc that contradicts its signature is ENCODE, with `instead:` the corrected sentence and the doctest or doc-lint that would have caught it (rung 3 or 5)
- fill-in instructions and section labels in a template or scaffold file, where the owning skill or generator states the same rule — that design was judged once, at the template's own review, not per diff
- anything the user explicitly asked to keep this round

## Output

Findings ONLY. No preamble, no summary of what the change does. One finding per comment:

```
- <KILL | ENCODE | SUPPRESSION | KEEP?> — <file:line> — <the comment, quoted>
  claims: <what the comment asserts about the code, as checkable sentences — one per claim>
  checked: <what you ran or read — the line's history, the grep for the referent, the rule's docs — and what it showed>
  instead: <KILL: delete | ENCODE: rung N — the design change, type, test, assertion or lint that holds it, or "rung unclear — needs the author" | SUPPRESSION: the scoped, reasoned, expiring form, or "fix the error" | KEEP?: the pointer that would earn it>
```

Then one line, `suppressions in touched files: <n> | none — swept for <directive families>`: a record that cannot show the sweep ran cannot tell "none" from "did not look".

A diff whose comments all sit on the floor: say so in one line and return an empty list.

**Append** this output to the review record you were given, under the header you were given, never touching an earlier section. If you cannot write it (no path given, no filesystem), hand the findings back and say they went unrecorded.

## Rules

- **No finding without a `checked:` line** — and no KILL of a why-comment, no KEEP?, without blame (`git log -L`) on the line first: its history names the incident it came from, and a fence has a builder you find before you pull it. Then grep the referent: the `keep in sync with foo.ts` whose `foo.ts` is gone is a KILL with proof; the one whose `foo.ts` is live is ENCODE at rung 3 or 5. A constraint comment where the **code** may be the wrong side (`must hold the lock`, and the code does not) is ENCODE whose `instead:` names the test that decides — say so; `reviewing-the-diff` owns what happens when that test fails.
- **You produce verdicts; you do not delete, encode, or edit.** Read-only over the code — your review record is the one file you may write, and nothing else, anywhere.
- **Review the comment, not the author.** A comment's age and its author's seniority are not evidence; its claim is.

## When you are the same session that wrote the diff

No subagent available? Run this as its own pass, third, after the `ponytail-review` pass. Your own comments all feel necessary — you wrote them to explain something — so do not ask whether each is justified; ask what each one *claims* and what *checks* it, and write the verdict before moving to the next. Name the record yourself, since no dispatcher did: `.ratchet/review/<task-id>-<lens>.md`. Deleting as you go is how the easy kills crowd out the encodings.
