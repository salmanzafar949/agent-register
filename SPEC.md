# Agent Register — Specification

Version 0.1 · 2026-10-03

Agent Register gives every project a register in plain files that any agent can read:
**tasks** (what to do), **artifacts** (what was produced), **memory** (what happened)
and **mistakes** (what not to repeat). `AGENTS.md` says how to work; `.register/`
holds everything else.

The words MUST, SHOULD and MAY are used as in RFC 2119.

## 1. Layout

```
project/
├── AGENTS.md              # how to work (in Git, 50–120 lines)
├── CLAUDE.md              # same block, or a line importing AGENTS.md
└── .register/
    ├── tasks/
    │   ├── index
    │   └── T-20261001-sz-1-e2e-flaky-login.md
    ├── artifacts/
    │   ├── index
    │   └── T-20261001-sz-1/retry-report.md
    ├── memory/
    │   ├── index
    │   └── T-20261001-sz-1.md
    └── mistakes/
        ├── index
        └── M-20260928-ak-1-token-expiry.md
```

`AGENTS.md` changes rarely and holds no history. Everything that changes lives in
`.register/`.

## 2. Storage

- **Default:** `.register/` is git-ignored and lives on the developer's machine.
  Teams sync it to shared storage (Azure Blob, Google Drive, OneDrive, S3) with a
  CLI they already use, such as `az`, `rclone` or `gsutil`.
- **Alternative:** commit `.register/` to Git. You get history, diffs and merge
  conflict resolution, at the cost of register changes appearing in pull requests.
  Do not commit it if it may hold secrets or personal data.

Whichever you choose, write the sync command into the AGENTS.md block (section 8).

## 3. IDs

IDs MUST NOT depend on reading a shared counter, so parallel agents do not collide.

| Kind | Format | Example |
| --- | --- | --- |
| Task | `T-<YYYYMMDD>-<initials>-<n>` | `T-20261001-sz-1` |
| Child task | `<parent id>.<n>` | `T-20261001-sz-1.1` |
| Mistake | `M-<YYYYMMDD>-<initials>-<n>` | `M-20260928-ak-1` |

- `<initials>` identifies the person the agent works for, lowercase.
- `<n>` starts at 1 and counts that person's IDs of that kind on that day. Pick the
  next number after the highest one in the index with the same prefix.
- File names are `<id>-<slug>.md`, with a short kebab-case slug.
- Every file carries its task ID, so an agent can follow a mistake to its task to
  its artifacts.

## 4. Indexes

Each folder has one `index` file: one line per entry, never details.

```
- T-20260928-ak-1 | auth token refresh | done | 2026-09-28 | tasks/T-20260928-ak-1-auth-refresh.md
- T-20261001-sz-1 | flaky e2e login test | in progress (codex) | 2026-10-01 | tasks/T-20261001-sz-1-e2e-flaky-login.md
```

Fields: `id | title | status | last updated | path`. The memory, artifacts and
mistakes indexes use the same shape; the status field holds the agent, the artifact
type or the mistake status respectively.

An agent MUST edit only the lines of its own task and MUST NOT rewrite or reorder
other lines. The task index is the first file an agent reads.

## 5. Files

Templates are in [`templates/`](templates/).

### Task — `tasks/<id>-<slug>.md`

```markdown
---
id: T-20261001-sz-1
title: Flaky e2e login test
status: in progress        # open | in progress | blocked | done | dropped
claimed_by: codex          # empty when unclaimed
claimed_at: 2026-10-01T09:12:00Z
tags: [e2e, auth]
updated: 2026-10-01
memory: memory/T-20261001-sz-1.md
artifacts: [artifacts/T-20261001-sz-1/retry-report.md]
mistakes: []
---
## Goal
What done looks like.
## Children
### T-20261001-sz-1.1 — retry wrapper
```

Follow-up work is added under **Children**. It never becomes a new task.

### Memory — `memory/<task id>.md`

```markdown
---
task: T-20261001-sz-1
agent: claude-code
updated: 2026-10-01
commit: 3f2a9c1             # HEAD when this memory was last written
---
## Done
## Decisions
## Files touched
## Next step
```

If the code has moved well past `commit`, treat the memory as possibly stale and
check it against the code before relying on it.

### Mistake — `mistakes/<id>-<slug>.md`

```markdown
---
id: M-20260928-ak-1
task: T-20260928-ak-1
tags: [auth]
date: 2026-09-28
status: active              # active | superseded | promoted
superseded_by:              # mistake id, when superseded
promoted_to:                # where the rule now lives: AGENTS.md, a test, a lint rule
---
## What happened
## How it was caught
## Root cause
## Fix
## Rule for next time
```

- **active:** agents load it and follow its rule.
- **superseded:** a newer mistake replaces it. Agents skip it.
- **promoted:** the rule now lives in AGENTS.md, a test or a lint rule, so agents
  no longer need to load it. This is the best outcome: a rule a machine enforces
  cannot be forgotten.

### Artifacts — `artifacts/<task id>/`

Anything a task produces: reports, generated files, outputs. List each one in the
task's `artifacts` field and in `artifacts/index`.

## 6. Claims

- An agent claims a task by setting `claimed_by` to itself and `claimed_at` to the
  current UTC time.
- A claim expires after **4 hours** without an update to the task or its memory.
  Teams MAY set a different limit in the AGENTS.md block.
- If the claim is live and held by someone else, the agent MUST stop and ask.
- If the claim has expired, the agent MAY take it over, and MUST say so in the
  task's memory file.
- On finishing, the agent clears `claimed_by` and `claimed_at`.

## 7. The agent loop

1. **Fetch.** Sync `.register/` from shared storage, if used.
2. **Find the task.** Read `tasks/index`. Open the task if it exists; otherwise
   create it with a new ID and add its index line. Claim it.
3. **Load context.** Read the task's memory file and every **active** mistake
   whose tags match the task. Follow each "Rule for next time".
4. **Work.** Save outputs under `artifacts/<task id>/`.
5. **Checkpoint.** At each milestone (a fix landed, a decision made, before a long
   step), update the memory file and the task's `updated` date. This refreshes the
   claim and means a crashed session still leaves a handoff.
6. **Write back.** At the end: update the task (status, artifacts, mistakes, clear
   the claim), finish the memory file with the current `commit`, add a mistake file
   if anything went wrong, update this task's index lines, then sync.

When the history is too large to read, step 2 becomes a search: read the index to
see what exists, then retrieve only the relevant files, filtered by tags.

## 8. The AGENTS.md block

Paste [`AGENTS-BLOCK.md`](AGENTS-BLOCK.md) into `AGENTS.md`, and into `CLAUDE.md`
for tools that read it. Fill in the sync command and your initials convention.
Keep the whole `AGENTS.md` between 50 and 120 lines.

## 9. Known limits

- **It relies on agents following instructions.** Long sessions or weaker models
  can skip a write-back. Checkpoints reduce the loss; hooks, where a tool supports
  them, make write-back automatic.
- **Indexes are shared files.** Two agents finishing at the same moment can
  overwrite each other's index line. Storage leases or ETags reduce this; Git
  merges help when `.register/` is committed.
- **IDs can still collide** if one person runs two agents in parallel on the same
  day. Give each agent its own initials suffix (for example `sz1`, `sz2`) when you
  do that.
- **Nothing is live.** An agent only sees changes when it fetches.
- **Search copies can be stale** if the shared copy is re-indexed only occasionally.
