---
name: retrospecting
description: Use after landing any Tier 2/3 task (mandatory, triggered by landing-the-change), after any Tier 0/1 task that produced a surprise, after any debugging session that took more than ~30 minutes, or when the user says "let's not make that mistake again" / "remember this for next time". Converts this task's friction into rules the NEXT session will actually load.
---

# Retrospecting

A lesson that lives in the conversation dies with the conversation. This skill mines the just-finished task for the small number of *project-specific, testable* rules that would have made it smoother, and writes them where every future session looks: `.ratchet/LESSONS.md`.

**Prevents:** failure mode #10 (repeating the same project-specific mistakes session after session).

**Not a sub-agent.** When `landing-the-change` parallelizes the record step it delegates the docs audit, but not you: you run in the main session, concurrently with that sub-agent. Picking the right lessons needs the friction the session actually lived through — not just what the worklog captured — and `LESSONS.md` is loaded by every future session, so what earns a line in it is a main-session judgment call.

## Step 1 — Mine the task record (not your impressions)

Re-read this task's worklog entries. The signal lives in:
- every `surprise` entry — what assumption broke? would a one-line warning have saved the detour?
- every `escalation` — was the original sizing wrong in a way that will recur for this kind of task here?
- the debugging trail — what did the root cause turn out to be, and what made it slow to find?
- gate/review findings — anything that nearly shipped?
- friction you remember that the record missed (note it now; also note the record missed it — that itself may be a lesson about worklog habits).

## Step 2 — Filter hard: most tasks yield ZERO lessons

A lesson earns its line in LESSONS.md only if ALL hold:
1. **Project-specific** — not general craft ("write tests" is the methodology's job, not the lessons file's).
2. **Recurring** — will plausibly bite again, in a recognizable situation.
3. **Actionable as a trigger → rule** — a future session can CHECK it, not just nod at it.
4. **Not already derivable** — from the codebase, README, or an existing lesson (if an existing lesson half-covers it, sharpen THAT one).

"We should be more careful with the config" fails (no trigger, no action). "The dev server caches `config/*.yaml`; restart it after any config edit or you'll debug a stale value" passes.

## Step 3 — Write the lesson

Append to `.ratchet/LESSONS.md`:

```markdown
## L7: Restart dev server after editing config/*.yaml
when: edit under config/
rule: restart dev server before trusting behavior — caches yaml at boot
because: 2026-06-09-oauth-login — 40 min debugging "stale" client id (worklog 14:20)
```

Format is binding: `when` is the recognizable trigger; `rule` is the checkable behavior; `because` cites task + worklog so a future skeptic can audit it. One lesson per heading, numbered sequentially.

**Write each line in caveman-full style — max signal per line.** Drop articles, filler, and hedging; fragments are fine; prefer the short word. Keep verbatim, never compress: code, paths, API names, CLI commands, flags, and exact error strings. One carve-out: if dropping a conjunction or reordering would make an order-sensitive `rule` misread ("do X *before* Y"), keep enough grammar to stay unambiguous — a lesson that misreads costs more than the saved tokens. LESSONS.md loads at every session start (Step 4); terse lines get followed, prose gets skimmed.

## Step 4 — Prune while you're here (the file must stay loadable)

LESSONS.md is read at EVERY session start (`using-ratchet` Step 0) — it competes for attention with the task itself. Keep it under ~30 lessons / ~150 lines:
- A lesson made obsolete (the flaky tool was replaced, the caching was fixed) → delete it.
- Two lessons with the same root → merge.
- A lesson that's really a code/docs fix ("the README's setup command is wrong") → fix the code/docs instead; that's a Tier 0 task, do it now, no lesson needed. **The lessons file is for what can't be fixed at the source.**
- A lesson that's general methodology, not this project → it doesn't belong here; if Ratchet's skills themselves have a gap, say so to the user (skills are improved deliberately, not via per-project notes).

## Step 5 — Close

1. Worklog `retro` entry: lessons added/merged/pruned (or "no lessons — clean task", which is a fine outcome).
2. Report to the user in one or two lines: what was learned, or that nothing recurring surfaced.

## Rationalization check

| Thought | Reality |
|---|---|
| "Nothing went wrong, skip the retro" | Maybe — Step 2 legitimately yields zero. But run Step 1 first: surprises hide in the worklog you've already mentally smoothed over. |
| "I'll write a few general reminders to be thorough" | Padding kills the file. Ten sharp lessons get followed; forty platitudes get skimmed. Zero is better than vague. |
| "This lesson is obvious in hindsight" | Hindsight-obvious is exactly what next session's cold start won't have. That's the test FOR writing it, not against. |
