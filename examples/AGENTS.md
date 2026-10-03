# Agents working on this project

## Closing work

- **A task is not done while any fix or bug is open.** Close it only when
  every review finding is fixed or explicitly accepted by the owner, tests
  prove it, and the result has been re-checked. Then move to the next task.
- **When unsure, ask. Never decide alone.** If the ask is unclear, or a
  change was not asked for, stop and ask before changing anything.
- Report what is still open plainly. Never call a task done with an open
  item hidden in the details.

## Bug fixing protocol

When a bug is reported, **do not jump to fixing it.**

1. **Reproduce.** Find the exact steps or conditions that trigger it.
2. **Root cause.** Trace the code path and find WHY it happens, not just where.
3. **Fix.** Only then change the code, targeting the root cause.

A change that hides the symptom without step 2 is not a fix. Record the root
cause in memory (see Memory below).

## Checks

Run all of these before calling any change done:

```bash
<backend tests>      # e.g. uv run pytest -q
<frontend tests>     # e.g. bun test
<typecheck>          # e.g. bunx tsc --noEmit (test runners often do not typecheck)
<build>              # e.g. bun run build
```

All of them, every time. Passing tests alone do not prove the app builds.

## Non-negotiables

These apply before any lookup would have happened.

- **Ports in use belong to the user.** Never kill a process to free a port.
  If you need a port, ask which one to use. Stop whatever you started when
  you are done.
- Never log or copy tokens, keys, connection strings, secrets or personal
  data into code, comments, docs, test fixtures or messages.
- Do not weaken security settings (auth, SSL, permissions) without a clear,
  agreed reason.
- Every UI change must be checked visually in the rendered view: spacing,
  sizes, alignment, states, overflow and responsive layout must match the
  surrounding UI. Passing tests do not excuse a visible inconsistency.
- Unrelated uncommitted changes belong to the user. Do not revert or
  overwrite them.

## Tests

- Write the test before the code, not after.
- Prefer end-to-end tests for real features, and leave a repeatable artifact
  (report, screenshot, log) that proves the result.
- When testing a part in isolation, first list the ways it can fail, then
  write the code.

## Skills

Working procedures live in `skills/<name>/SKILL.md`, plain markdown that any
agent can read. Check them before any general-purpose skill; a project skill
wins over a generic one. Read the relevant `SKILL.md` **in full** before
acting on a task it covers.

| Skill | Read it when |
|---|---|
| `skills/architecture/` | starting out, or deciding where a change belongs |
| `skills/development/` | changing code, migrations or dependencies |
| `skills/testing/` | running or writing tests, or before calling a change done |

**Creating a skill.** Ask one question: *could an agent break this rule
without ever thinking to look it up?* If yes, the rule belongs in this file.
If no, make it a skill. Each skill has one trigger, a `description` that
names the moment to use it (not a summary of its steps), a row in the table
above, and tool-neutral wording.

## Memory

All memory lives in `.memory/` at the repo root. It is git-ignored: local
working context that never enters repo history.

```text
.memory/
  MEMORY.md        index: one line per memory, read first
  <prefix>-<slug>.md   one subject per file
```

- **Start of session:** read `.memory/MEMORY.md`, then the entries relevant
  to the task. Create the folder and an empty index if missing.
- **After meaningful work:** update the entry, then its one index line.
  Never put details in the index, and never replace it; only append.
- **One entry per subject, not per session.** Search first; if an entry
  exists, append to it. Keep exactly one index line per subject.
- **Naming:** `project-`, `bug-`, `feature-`, `reference-`, `feedback-` or
  `archive-` plus a kebab-case slug named after the thing, never `notes`,
  `fixes` or a bare date. Link related entries with `[[entry-name]]`.
- **Content:** branch, commit, files touched, root cause, fix, how it was
  validated, and what is still open. Never rewrite history; append a
  correction instead.
