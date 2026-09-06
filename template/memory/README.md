---
name: memory-root-ownership
description: Declares this directory as the single source of truth for project memory, and the rules for reading it.
metadata:
  type: system_rule
---

# Memory Root

`{{MEMORY_ROOT}}` is the **single source of truth** for project memory.

> Required for Codex (which has no designated memory directory, so ownership must
> be declared). Harmless and useful on Claude Code — it documents the boundary
> for anyone, human or agent, who opens the folder later.

## Daily rules

- Read and write current project memory **only here**.
- **Start from `MEMORY.md`**, then open only the files the chosen project points
  to. Never load the whole tree.
- `commands/save-progress.md` is the **only** `/save-progress` procedure. If you
  find another copy elsewhere, it is stale.
- Keep `logs/` closed unless the user names a date, or the index genuinely cannot
  answer the question.
- Write durable daily progress to `logs/YYYY-MM-DD.md`. **Logs are not the source
  of current project state** — `state.md` is.

## Layout

```
{{MEMORY_ROOT}}/
├── README.md          ← this file
├── MEMORY.md          ← the index; always read first
├── user_profile.md
├── feedback_*.md      ← behavioural rules (or a feedback/ subdirectory)
├── projects/
│   └── {p}/           ← core.md · state.md · archived/
└── logs/              ← YYYY-MM-DD.md + archive.md
```

> With more than ~8 feedback files, move them into a `feedback/` subdirectory.
> The index lines in `MEMORY.md` are what matter; the folder shape is free.

## Running more than one agent

If you run a second agent (Codex alongside Claude Code, or vice versa), give each
one **its own memory root** and declare the boundary here. Two agents writing the
same tree will silently overwrite each other's state.

- **This root is owned by**: {{AGENT_NAME}}
- **Other roots are read-only fallback**: `{{OTHER_ROOT}}`

Read the other root only when:

- the user explicitly asks for context from that agent's era;
- this root lacks the historical context required;
- a migration, recovery, audit, or compatibility task requires it.

**Never write to another agent's root** unless the user explicitly asks.

When a project moves between agents, write a handoff file that points at the new
source of truth, and mark the project 📦 in the old index rather than deleting it.
