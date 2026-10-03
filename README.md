# Agent Register

**Shared memory for coding agents, in plain files.** Tasks, artifacts, memory and
mistakes that Claude Code, Codex, Cursor, DeepSeek or any other agent can read,
so context survives between sessions, between tools and between teammates.

## The problem

Every coding agent starts a session with no idea what happened yesterday. You
re-explain the project, the agent re-discovers decisions, and the same mistakes
come back. Switch tools and it gets worse: each vendor's memory lives inside its
own product. In a team, your agent has no idea what a teammate's agent just did.

## Why this is better

| Approach | What goes wrong | Agent Register |
| --- | --- | --- |
| History stuffed into AGENTS.md / CLAUDE.md | Bloats, goes stale, agents follow outdated rules | AGENTS.md stays short and stable; history lives in `.register/` |
| Vendor memory (built into one tool) | Doesn't follow you to another tool or teammate | Plain files any agent can read |
| Paid memory services | Per-call cost, opaque to humans, tied to one stack | No SDK, no account, no subscription |
| Re-reading chat transcripts | Slow, expensive, mostly noise | Read a one-line index, open only what matters |
| Task trackers in markdown | Track work, not what was learned | A dedicated **mistakes** layer: root cause plus a rule the next agent follows |

- **Zero tooling.** Folders, markdown and a short block of instructions.
- **Vendor-neutral.** Proven handoffs between Claude Code, Codex and DeepSeek.
- **Mistakes are first-class.** Each failure is recorded with a root cause and a
  rule, linked to its task, and loaded automatically for matching work.
- **Built for parallel agents.** One ID per task; IDs never come from a shared
  counter; agents edit only their own task's lines.
- **Works solo or as a team.** Local folder alone; sync it to shared storage for a
  team.

## How it works

```
project/
├── AGENTS.md          # how to work (short, in Git)
└── .register/
    ├── tasks/         # one file per task: goal, status, claim
    ├── artifacts/     # what each task produced
    ├── memory/        # what was done and decided, and the handoff
    └── mistakes/      # what went wrong, root cause, rule for next time
```

Every agent follows the same loop: **fetch → find the task → load its memory and
matching mistakes → work → checkpoint → write back.**

## Quick start

1. Create the folders:
   ```bash
   mkdir -p .register/{tasks,artifacts,memory,mistakes}
   for d in tasks artifacts memory mistakes; do
     printf '# id | title | status | last updated | path\n' > .register/$d/index
   done
   ```
2. Paste [`AGENTS-BLOCK.md`](AGENTS-BLOCK.md) into your `AGENTS.md` (and
   `CLAUDE.md`). Fill in your initials and sync command.
3. Add `.register/` to `.gitignore`, or commit it (see the spec, section 2).
4. Ask your agent to start a task.

## Contents

- [`SPEC.md`](SPEC.md): the full specification
- [`AGENTS-BLOCK.md`](AGENTS-BLOCK.md): the block to paste into AGENTS.md
- [`templates/`](templates/): task, memory, mistake and index templates
- [`examples/AGENTS.md`](examples/AGENTS.md): a real, trimmed AGENTS.md

## Roadmap

1. **Open spec** (this repo).
2. **CLI:** `register init`, `task`, `sync`, `mistake`, `check`. Enforces IDs,
   prevents duplicates, keeps indexes consistent.
3. **Live task tracker for agents:** cards, claims with expiry, notifications when
   a card changes, lessons handed to an agent when it claims matching work.
   Self-hostable and open source.

## Related work

- [AGENTS.md](https://agents.md): the instruction file this extends.
- [Cline Memory Bank](https://docs.cline.bot/best-practices/memory-bank.md):
  single-tool memory files, no mistakes layer.
- [Backlog.md](https://github.com/MrLesk/Backlog.md): tasks as markdown; tracking,
  not memory or lessons.
- [beads](https://github.com/steveyegge/beads): a Git-backed issue tracker for
  agents; needs its own binary.
- Mem0, Letta, Zep: API memory for apps, not files shared between coding agents.

## License

[MIT](LICENSE)
