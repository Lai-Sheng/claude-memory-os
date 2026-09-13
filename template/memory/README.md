---
name: memory-root-ownership
description: Declares this directory as the single source of truth for project memory, and the rules for reading it.
metadata:
  type: system_rule
---

# Memory Root

`{{MEMORY_ROOT}}` is the **single source of truth** for project memory.

> This file documents the boundary for anyone, human or agent, who opens the
> folder later — including you in six months.

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
- **Only write here when the user runs `/save-progress`**, or explicitly asks you
  to remember something. A session that was never saved is supposed to be gone.

## Why one root for every project

The router opens each session with a menu of **all** active projects. That only
works if every project is visible from one index. Splitting memory per folder
would give isolation for free and lose the menu; this architecture keeps the menu
and builds isolation instead (core/state, zoning, freezing). See
`docs/architecture.md → One memory root, on purpose`.

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

## Handing a project off

When a project moves to someone (or something) else, do not delete it and do not
keep maintaining a copy. Write a short handoff note that points at the new source
of truth, mark the project 📦 in `MEMORY.md`, and drop it from the menu. From then
on it is outside this root's responsibility — not a gap in it.
