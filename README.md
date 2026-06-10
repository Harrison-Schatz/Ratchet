# Ratchet

**Evidence-gated progress that survives interruption.**

Ratchet is a software development methodology for coding agents, packaged as 19 composable skills. A ratchet moves freely forward and never slips back: each click is a verified step, and the pawl that holds it is *evidence on disk* — not the agent's memory, not the conversation, not a confident claim.

It is built on two observations:

1. **Agent-driven projects fail in predictable ways** — building the wrong thing, silent scope creep, completion theater, lost context between sessions, confident wrong fixes, plans that quietly drift from reality.
2. **The conversation is the worst possible place to keep the truth.** Sessions die, context compacts, and the next agent starts cold. Anything that matters must live on disk.

Every skill in this catalog exists to defeat a *named* failure mode. A skill that prevented nothing was deleted.

---

## How it works in 30 seconds

Every task — typo to subsystem — walks the same five beats. Only the middle inflates.

```
ORIENT → SIZE → BUILD (tier-scaled) → VERIFY → RECORD
```

A **typo fix** costs ~30 seconds of sizing, one fresh verification command, and a commit. No questions, no design doc, no ceremony.

A **new feature touching auth** is forced to Tier 2: a confirmed brief with observable acceptance checks, a step-by-step plan-as-hypothesis, checkpointed execution where every step ends committed and recorded, an independent review (with a second reviewer pass on the auth files), an evidence-gated "done," and a retrospective that writes lessons the next session will load.

The difference between those two paths is decided by **rules, not vibes** — and a "small" task that reveals itself to be large escalates mid-flight for the cost of one log entry.

## Why another methodology?

Most agent process frameworks have one pipeline and one speed. In practice that means heavyweight ceremony gets applied to one-line fixes until the agent (or the human) starts skipping steps — unpredictably, and usually the wrong ones. Ratchet's bet is different:

| Problem with one-speed process | Ratchet's answer |
| --- | --- |
| A config change pays the same tax as a subsystem | **Four tiers** with objective sizing criteria; ceremony must pay rent |
| Crash mid-task loses everything the agent knew | **`.ratchet/STATE.md`** — a fresh session resumes by *reading*, not reconstructing |
| Plans rot the moment reality disagrees | **Plans are hypotheses** with a change log and a defined replan protocol |
| "Done!" without proof | **Structural gate**: no completion claim without a freshly-run command and its output on record |
| Legacy code with no tests gets apologies, not workflows | **Brownfield is first-class**: characterization tests and a six-move seam catalog |
| The same project-specific mistake, every session | **`LESSONS.md`**: retrospectives write `when / rule / because` entries every future session loads |

## The failure modes

Each maps to a concrete, checkable mechanism — never a value statement. The full table with mechanisms is in [METHODOLOGY.md](METHODOLOGY.md).

1. Building the wrong thing
2. Silent scope creep
3. Completion theater
4. Untested happy path
5. Context loss between sessions
6. Confident wrong fixes
7. Plans that drift from reality
8. Breaking legacy code
9. Mis-sized process
10. Repeating project-specific mistakes
11. Documentation drift

## The tiers

| | Tier 0 — Patch | Tier 1 — Task | Tier 2 — Feature | Tier 3 — Project |
| --- | --- | --- | --- | --- |
| **Looks like** | typo, rename, config tweak | one subsystem, ≤~5 files, unambiguous | new components, ambiguity, new deps, or any risk surface* | multi-session, needs decomposition |
| **Process** | just do it | Definition of Done → test-first → gate | brief → plan → checkpointed execution → review → gate → retro | Tier 2 per milestone |
| **Human gates** | none | none | brief approval; merge | brief + decomposition; per-milestone |

\* **Risk surfaces** (any one forces minimum Tier 2): auth/authz, payments, data migration or deletion, secrets, concurrency primitives, public API contracts.

## The state directory

Everything a fresh session needs lives in `.ratchet/` at the repo root:

```
.ratchet/
├── STATE.md      # current snapshot — task, tier, phase, step, NEXT ACTION (one imperative sentence)
├── WORKLOG.md    # append-only journal: sizings, decisions, surprises, evidence, escalations
├── LESSONS.md    # project rules earned from retrospectives, loaded at every session start
├── briefs/       # confirmed intent: goals, observable acceptance checks, out-of-scope lists
└── plans/        # step sequences with proofs and a change log
```

If the session dies mid-task, the cost is at most one step: the next agent reads STATE.md, verifies it against `git status` and the test suite (state is a *claim* until checked), reconciles, and continues.

## The skill catalog

Load **`using-ratchet`** first — it routes every request and makes the rest predictable.

**The spine**
| Skill | Defeats |
| --- | --- |
| [`sizing-the-task`](skills/sizing-the-task/SKILL.md) | mis-sized process; the escalation escape hatch |
| [`writing-the-brief`](skills/writing-the-brief/SKILL.md) | building the wrong thing |
| [`planning-the-work`](skills/planning-the-work/SKILL.md) | executor ambiguity; drift |
| [`executing-with-checkpoints`](skills/executing-with-checkpoints/SKILL.md) | context loss; silent improvisation |
| [`verifying-done`](skills/verifying-done/SKILL.md) | completion theater; scope creep at the exit |
| [`landing-the-change`](skills/landing-the-change/SKILL.md) | work declared done but never integrated |
| [`retrospecting`](skills/retrospecting/SKILL.md) | repeating project-specific mistakes |

**Cross-cutting**
| Skill | Defeats |
| --- | --- |
| [`keeping-state`](skills/keeping-state/SKILL.md) / [`resuming-work`](skills/resuming-work/SKILL.md) | context loss between sessions |
| [`replanning-on-surprise`](skills/replanning-on-surprise/SKILL.md) | plans that drift from reality |
| [`testing-by-default`](skills/testing-by-default/SKILL.md) | untested happy paths |
| [`characterizing-legacy-code`](skills/characterizing-legacy-code/SKILL.md) / [`finding-seams`](skills/finding-seams/SKILL.md) | breaking legacy code |
| [`debugging-to-root-cause`](skills/debugging-to-root-cause/SKILL.md) | confident wrong fixes |
| [`delegating-to-agents`](skills/delegating-to-agents/SKILL.md) | subagent completion theater |
| [`reviewing-the-diff`](skills/reviewing-the-diff/SKILL.md) | scope creep; late intent misses |
| [`prototyping-to-decide`](skills/prototyping-to-decide/SKILL.md) | guesses baked into briefs and plans |
| [`syncing-the-docs`](skills/syncing-the-docs/SKILL.md) | documentation drift |

Each SKILL.md is 50–90 lines: pushy trigger description, numbered steps, explicit outputs and stop conditions, and a short rationalization table where the discipline needs armor. Depth lives in `references/` files loaded only when needed.

## Getting started

The skills use the standard `SKILL.md` format (YAML frontmatter + body) and work with any agent harness that supports it (Claude Code and compatibles).

1. Copy the `skills/` directories into your harness's skills location (e.g. `.claude/skills/` for a project, or your user-level skills directory).
2. Ensure `using-ratchet` loads at session start — via your harness's session hook, or by listing it first in your project instructions (`CLAUDE.md`/`AGENTS.md`).
3. Work normally. The first Tier 1+ task creates `.ratchet/`; commit it — state that isn't pushed dies with the laptop.

There is deliberately **no dependency on hooks or tooling**: even if nothing is injected, `.ratchet/STATE.md` sitting in the repo root is discoverable by any agent that lists the directory. The methodology degrades gracefully instead of failing silently.

## What Ratchet refuses to do

- **Interview you about a typo.** Tier 0 exists so the spine costs seconds when the task warrants seconds.
- **Write code twice.** Plans specify intent, files, and proof per step — never pre-written implementations.
- **Block on theatrical approval.** Human gates sit where decisions are irreversible or intent is unknowable — not at every phase boundary.
- **Moralize.** Gates are structural ("the claim requires the evidence line"), not rhetorical. Rules that matter are few, so they stay loud.
- **Depend on memory** — the agent's or the model's. Disk beats conversation.

## Provenance

Ratchet was designed by deeply studying [obra/superpowers](https://github.com/obra/superpowers), keeping what survived an adversarial critique (the verification gate, root-cause debugging, zero-context plan readability, fresh-context delegation, descriptions-as-triggers, progressive disclosure) and rebuilding the rest from first principles. The receipts are in this repo:

- [`analysis/superpowers-map.md`](analysis/superpowers-map.md) — full map of the studied system: every skill, the flow graph, the implicit design principles
- [`analysis/critique.md`](analysis/critique.md) — the adversarial case: ceremony costs, under-specification, baked-in assumptions, and the compliance-vs-outcome evidence gap
- [`METHODOLOGY.md`](METHODOLOGY.md) — the manifesto: failure modes, mechanisms, tiers, principles, and every divergence with its stated reason
- [`stress-tests.md`](stress-tests.md) — five scenarios traced skill-by-skill (typo fix, OAuth in an untested Express app, ambiguous greenfield CLI, mid-plan disaster, dead-session resume), including the ambiguities the traces exposed and the fixes applied

Every divergence from the original has a written reason. Every skill answers "which failure mode does this prevent?" If you can't predict which skill fires for a given request from METHODOLOGY.md and `using-ratchet` alone, that's a bug — file it.

## Repository layout

```
METHODOLOGY.md      # the one-page manifesto — start here
skills/             # the 19 skills (SKILL.md + optional references/)
analysis/           # the study and critique that produced the design
stress-tests.md     # the five dry-run traces
```

## License

MIT
