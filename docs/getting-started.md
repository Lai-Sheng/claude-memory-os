# Getting Started

15 minutes to a working setup.

## 0. Where things go

Claude Code reads user-level config from `~/.claude/`:

```
~/.claude/
├── CLAUDE.md              ← L1 router (always loaded)
├── commands/              ← your slash commands
│   ├── save-progress.md
│   └── commands.md
└── projects/<workspace>/memory/
    ├── MEMORY.md          ← L2 index (always loaded)
    ├── user_profile.md
    ├── feedback_*.md
    ├── projects/          ← L3 per-project core/state
    └── logs/              ← L4 chronological
```

> On Windows the path is `C:\Users\<you>\.claude\`.
> Confirm your exact memory directory — Claude Code will tell you where it is if
> you ask it *"where is your memory directory?"*.

## 1. Copy the templates

```bash
git clone https://github.com/Lai-Sheng/claude-memory-os.git
cd claude-memory-os

cp template/CLAUDE.md            ~/.claude/CLAUDE.md
cp template/commands/*.md        ~/.claude/commands/
cp -r template/memory/*          ~/.claude/projects/<workspace>/memory/
```

⚠️ If you already have a `CLAUDE.md`, **merge** rather than overwrite — the
template will not know your existing rules.

## 2. Fill in the placeholders

Every `{{PLACEHOLDER}}` needs a real value or a deletion. Start with:

1. **`CLAUDE.md` §1 Persona** — or delete the section entirely for a neutral
   assistant
2. **`CLAUDE.md` §2 Router** — list the projects you actually have right now.
   Two is a fine start
3. **`memory/user_profile.md`** — role, stack, working context. Keep it stable;
   anything that changes monthly belongs in a project `state.md`

Leave §4 (hard rules) mostly empty at first. Rules should come from real
incidents, not from imagination.

## 3. Create your first project

```bash
cd ~/.claude/projects/<workspace>/memory/projects
cp -r _project_template t1_my_project
```

Fill in `core.md` — especially **"Why this project exists"** and **"Definition of
done"**. Leave `state.md` mostly empty; the first `/save-progress` will populate it.

Add one line to `MEMORY.md`:

```markdown
- [T1 My Project](projects/t1_my_project/core.md) — 🛠️ **2026-01-15**: purpose · key constraint
```

Link `core.md` only. Never link `state.md` from the index.

## 4. Use it

Start a conversation. The assistant should open with the project menu. Pick one,
work, and before you close the terminal:

```
/save-progress
```

It measures your files, warns you if anything is over budget, overwrites
`state.md`, writes today's log, and tells you where to resume.

## 5. Let rules accumulate

When you correct *how* the assistant works — not what it built — write a feedback
file:

```markdown
---
name: feedback-something
description: One line, used to judge relevance at recall time
metadata:
  type: feedback
---

The rule.

**Why:** the incident.
**How to apply:** concretely, what changes.
```

Index it in `MEMORY.md`. Six of these is a system that fits you; sixty is bloat —
merge and prune when they overlap.

---

## Sanity checks

Run these occasionally:

| Check | Command | Limit |
|---|---|---|
| Index size | `wc -c MEMORY.md` | 16 000 bytes |
| Router size | `wc -c CLAUDE.md` | ~6 000 bytes |
| Biggest project file | `ls -S projects/*/core.md \| head -1` | 16 000 bytes |
| Any `state.md` linked from the index? | `grep state.md MEMORY.md` | should be empty |

## Common mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| Treating `CLAUDE.md` as a manual | It passes 10 KB, replies get generic | Move content to `memory/`, keep the router |
| Appending to `state.md` | It grows every session | Overwrite; push old state to `archived/` |
| Copying logs into project files | Same text in two places, one goes stale | The log is the history; the project file is the present |
| Linking `state.md` from the index | Context cost never drops after splitting | Link `core.md` only |
| Writing rules with no "why" | Rules get ignored or cargo-culted | Every rule records its incident |
