---
name: vsa-review
description: Use to review a diff for code placed outside its slice, imports that cross slice boundaries, and slices that change as one, and hand back MOVE, CROSS and BOUNDARY verdicts — run on every review round, or standalone when the user says "slice review", "VSA review", "is this in the right slice", "check the feature boundaries", or asks whether a change belongs in features/, shared/ or platform/. Works on any language, in a repo already organized by slices or a layered repo moving to them.
---

# Vertical Slice Architecture Review

You are the zoning inspector. You ask one question — **does this code live with the code it changes with, and does it reach across a boundary it should not?** — not whether it works, not whether it should exist.

This project is organized by vertical slices. A layered repo is a repo mid-migration: every new capability goes in a slice.

**Prevents:** failure mode #2 at the structure level — a change that leaks past its capability into code the author never examined; and #13, structural drift — new work that extends the layers the project has committed to replace.

## Scope

Every file added, moved, or deleted in `BASE..HEAD`; the **imports and entry points of every touched file, whole** — the import that breaks a boundary sits at the top of the file, not in the hunk; the co-change of each pair of slices the diff touches. Anything else is pre-existing: at most one line, labelled as such, never a finding.

**Skip by rule** when every touched file is on the floor: write `skipped — <n> files, all floor` and return nothing else.

## The standard

```
A slice is one business capability, end to end, in features/<capability>/
(or the repo's own spelling: modules/, slices/; infra/ for platform/):
- it owns the entry points (routes, commands, jobs, event handlers) that serve that capability
- it holds everything only those entry points need: handlers, validation, queries, models, mapping
- each use case is one file, holding one entry point and the handler behind it
- it is named for the capability in the business's words (invoicing, user-signup),
  never for a technical role (utils, helpers, common, services, managers)
```

## The ladder

For each unit of code the diff adds or moves, find the lowest rung that fits. Code anywhere else is a MOVE.

```
1. Serves a capability a slice owns?             → that slice
2. Plumbing with no business rule?               → platform/  (db connection, logging, auth middleware, config, clients)
3. A second slice needs the same logic?          → a copy in the second slice — never an import
4. 3+ slices hold copies that changed together
   for the same reason?                          → shared/, named for the business concept
5. A capability no slice owns?                   → a new slice, features/<capability>/ (the first one founds features/)
```

## The boundary rules

```
features/<a>        ──✗──▶  features/<b>   a slice never imports another slice
features/<a>        ──✓──▶  platform/, shared/
platform/, shared/  ──✗──▶  features/
composition root    ──✓──▶  features/      registers routes, commands, jobs — a registration is not an entry point
legacy layers       ──✗──▶  features/      old code never depends on new slices
```

## The metric: are two slices one?

Only when the diff touches two or more slices; for each pair:

```
git log -n 20 --format=%h --name-only --full-diff -- features/<a> features/<b>
```

Drop every commit that touches 4+ slices — a sweep, not a coupling. Fires when 6 or more commits remain and more than half touch both slices. Fewer than 6 → `insufficient history`, no claim.

The opposite failure — one slice that should be two — has no reliable signal yet.

## The verdicts

- **MOVE** — wrong rung. Name the rung and the target path. Covers: capability logic in `shared/` or `platform/`; a `shared/` extraction with fewer than three real consumers; a new capability added to a legacy layer; a slice named for a technical role; an entry point the diff adds to a file that already holds one (target `features/<slice>/<use-case>.<ext>`).
- **CROSS** — a touched file imports across a boundary rule. Name the import, then the lowest fix: copy into the slice (rung 3), move to `platform/` (rung 2), or extract to `shared/` (rung 4, its three consumers listed).
- **BOUNDARY** — the metric fired: two slices change as one. **Never blocks this diff** — restructuring slices inside an unrelated change is the scope creep this lens exists to stop. Recorded for a follow-up task.

## The floor — never a finding

- layered code changed in place — migrating it is a separate task the user asks for
- locations a framework or tool mandates: migration folders, generated code, package manifests, CI config
- database schema and migrations — one schema serves every slice
- where tests live
- the inside of a slice, beyond the standard: file count, file size, patterns, names
- anything the user explicitly asked for this round

## Output

Findings only — no preamble, no tour of the architecture:

```
- <MOVE | CROSS> — <file:line> — <what sits where, or what imports what, one sentence>
  rung: <1–5>
  checked: <what you ran or read, and what it showed>
  instead: <MOVE: target path | CROSS: rung 2, 3 or 4 fix with target>

- BOUNDARY — features/<a> + features/<b> — follow-up
  metric: co-change <both>/<remaining> over 20 commits (<hashes>)
  instead: merge into features/<x>/
```

Then the sweep block, always — nothing found means the block alone:

```
slice root:   <path | features/ (founded by this diff)>
imports:      <n cross-slice | none> — swept for <forms>; read by hand: <wiring a grep cannot see | none in this repo>
entry points: <files with 2+ | none> — swept for <forms>; read by hand: <convention routing | none in this repo>
co-change:    <pair: ratio | insufficient history> | n/a — one slice touched
```

A record that cannot show the sweep ran cannot tell "none" from "did not look".

**Append** this output to the review record you were given, under the header you were given, never touching an earlier section. If you cannot write it, hand the findings back and say they went unrecorded.

## Rules

- **No finding without evidence.** A MOVE or CROSS carries `checked:`; a rung-4 claim lists its three consumers by path; a BOUNDARY carries its hashes and the threshold it crossed.
- **Sweep, and name what you swept — and what a sweep cannot see.** Sweep imports in every form the touched files use, and entry points by definition. Where the repo wires either in a way a grep cannot see — path aliases, DI, convention routing — read the wiring by hand and say so in the sweep block.
- **Read-only over the code.** Your review record is the one file you may write.
- **Review the placement, not the author.**
