# Logic Prototypes — the terminal-app shape

Read this when the spike's question is about **business logic, state transitions, or data shape** — the kind of thing that looks reasonable on paper but only feels wrong once you push it through real cases. The artifact is a tiny interactive terminal app the user drives by hand.

Right-shape signals: "not sure this state machine handles X then Y", "does this data model represent the case where…", "I want to feel out the API before writing it", anything where the user should **press buttons and watch state change**. If the question is "what should this look like" — wrong shape; use `ui-prototypes.md`.

## 1. State the question

The spike declaration (SKILL.md Step 0) already names it. Repeat it in a comment at the top of the prototype's entry file so the code itself can be checked against its question later.

## 2. Pick the language

Whatever the host project uses; match its existing tooling. No new package managers or runtimes for a throwaway. No obvious runtime (docs repo)? Ask.

## 3. Isolate the logic in a portable, pure module

The logic answering the question lives behind a small **pure** interface that could be lifted into the real codebase later. The TUI around it is the throwaway part; the logic module is the potentially-graduating part. Pick the shape that fits the question, not the one easiest to wire to a terminal:

- **Pure reducer** — `(state, action) => state`. Discrete events, single state value.
- **Explicit state machine** — when "which actions are even legal right now" is part of the question.
- **Small set of pure functions** over a plain data type — when there's no ongoing state, just transformations.
- **Class/module with a clear method surface** — when the logic genuinely owns internal state.

Keep it pure: no I/O, no terminal escapes, no `console.log` for control flow. The TUI imports it and calls in; nothing flows the other way. A reducer that references the terminal is no longer portable — and portability is the only reason this code might outlive the spike.

## 4. Build the smallest TUI that exposes the state

Lightweight full-frame re-render: on every action, clear the screen (`console.clear()` / `print("\033[2J\033[H")`) and redraw the whole frame — one stable view, not growing scrollback. Each frame, in order:

1. **Current state**, pretty-printed and diff-friendly (one field per line or formatted JSON). Bold field names, dim secondary context (IDs, timestamps, derived values). Raw ANSI is fine (`\x1b[1m` bold, `\x1b[2m` dim, `\x1b[0m` reset) — no styling library unless the project already has one.
2. **Keyboard shortcuts** at the bottom: `[a] add user  [d] delete user  [t] tick clock  [q] quit`.

Loop: initialize in-memory state → render → read one keystroke/line → dispatch to a handler → re-render → until quit. The whole frame fits on one screen.

## 5. One command to run

Add a script to the project's task runner (`package.json` scripts, `Makefile`, `justfile`, `pyproject.toml`). No task runner → put the exact command in the spike declaration and at the top of the file.

## 6. Hand it over

The user drives it. "Wait, that shouldn't be possible" and "huh, I assumed X" are the bugs in the *idea* — the entire point. They want new actions? Add them; prototypes evolve within their one question.

## 7. Capture and clean up

The answer goes in the worklog spike-closed entry (SKILL.md Step 3) — not a NOTES.md that nobody reloads. Then delete the prototype, or graduate ONLY the pure module via `testing-by-default`'s spike-then-stabilize rules. The TUI shell always dies.

## Anti-patterns

- **Tests on the prototype.** A prototype needing tests is no longer a prototype — and the spike declaration is what made testlessness legal.
- **Wiring to the real database.** In-memory unless persistence IS the question.
- **Generalizing.** No "support X later." One question.
- **Blurring logic and TUI.** Terminal code in the reducer kills the one salvageable artifact.
- **Shipping the shell.** It was optimized for being driven by hand; production code it is not.
