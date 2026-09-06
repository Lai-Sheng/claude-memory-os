# Getting Started

15 minutes to a working setup.

> **On Codex?** The memory layers are identical, but the router file and one
> bootstrap step differ. Read **[codex.md](codex.md)** instead of §0–§1 here,
> then rejoin at §2.

## 0. Where things go

Two locations. The router lives where Claude Code reads config; the memory tree
lives **wherever you choose**, because you declare its path in the router.

```
~/.claude/
├── CLAUDE.md              ← L1 router
└── commands/              ← your slash commands
    ├── save-progress.md
    └── commands.md

~/AgentMemory/             ← memory root — any path you like
├── README.md              ← ownership declaration
├── MEMORY.md              ← L2 index
├── user_profile.md
├── feedback_*.md
├── projects/              ← L3 per-project core/state
└── logs/                  ← L4 chronological
```

> On Windows: `C:\Users\<you>\.claude\` and `C:\Users\<you>\AgentMemory\`.
>
> You *can* point the memory root at Claude Code's own project directory
> (`~/.claude/projects/<workspace>/memory/`; find yours with
> `ls -d ~/.claude/projects/*/`). It is not required, and a path you chose is
> easier to back up and to move between machines.

## 1. Copy the templates

```bash
git clone https://github.com/Lai-Sheng/claude-memory-os.git
cd claude-memory-os
```

```bash
mkdir -p ~/AgentMemory ~/.claude/commands
cp template/CLAUDE.md     ~/.claude/CLAUDE.md
cp template/commands/*.md ~/.claude/commands/
cp -r template/memory/.   ~/AgentMemory/
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force ~\AgentMemory, ~\.claude\commands | Out-Null
Copy-Item template\CLAUDE.md ~\.claude\CLAUDE.md
Copy-Item template\commands\*.md ~\.claude\commands\
Copy-Item -Recurse -Force template\memory\* ~\AgentMemory\
```

⚠️ If you already have a `CLAUDE.md`, **merge** rather than overwrite — the
template will not know your existing rules.

## 2. Fill in the placeholders

Every `{{PLACEHOLDER}}` needs a real value or a deletion. Start with:

1. 🚨 **`CLAUDE.md` §0 Memory Bootstrap** — the absolute path to your memory root.
   **Do this first, and do not delete the section.** It is what makes the memory
   layer load. Set the same path in `~/AgentMemory/README.md`
2. **`CLAUDE.md` §1 Persona** — or delete the section entirely for a neutral
   assistant
3. **`CLAUDE.md` §2 Router** — list the projects you actually have right now.
   Two is a fine start
4. **`memory/user_profile.md`** — role, stack, working context. Keep it stable;
   anything that changes monthly belongs in a project `state.md`

Then check nothing is left blank:

```bash
grep -rn "{{" ~/.claude/CLAUDE.md ~/AgentMemory/
```

Every line it prints is a placeholder you still owe.

Leave §4 (hard rules) mostly empty at first. Rules should come from real
incidents, not from imagination.

## 3. Create your first project

```bash
cd ~/AgentMemory/projects
cp -r _project_template t1_my_project
```

```powershell
Copy-Item -Recurse ~\AgentMemory\projects\_project_template ~\AgentMemory\projects\t1_my_project
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

🚨 **This is not an optional convenience. It is the whole enforcement loop.**

Every budget in this architecture is enforced by that command and by nothing
else. Skip it and `state.md` never gets overwritten, the index never gets rolled,
nothing ever moves to `archived/` — and in about six weeks you arrive at exactly
the bloated single file this repo exists to prevent, with better documentation.

Run it at the end of every session that changed anything. If you will not build
that habit, this architecture will not help you, and a plain `CLAUDE.md` is the
honest choice.

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

## Verify it actually works

Copying files proves nothing. The failure mode you are checking for is **silent**:
the files are all present and the agent never reads them.

Start a fresh session and ask about a project you have **not** mentioned in that
session — something only `state.md` knows, like *"where did I leave T1?"*

Then check the transcript:

| Check | Pass | Fail means |
|---|---|---|
| Did it **read a file**? | A visible read of `MEMORY.md` | §0 is not landing. Confirm you edited the installed router (`~/.claude/CLAUDE.md`), not the repo copy, and that `{{MEMORY_ROOT}}` is an absolute path that exists |
| Did it read **only** what it needed? | `MEMORY.md`, then that project's files | It is loading the whole tree — §0's "open only" clause was weakened or deleted |
| Did it open `logs/`? | It should **not** have | Reword §0 step 3; logs are for named dates only |
| Did the session **open with the project menu**? | Yes, before anything else | §2 Router is still placeholders, or your greeting was answered directly — check §2 lists real projects |
| Does it answer from `state.md`, or improvise? | Quotes your actual resume point | The file is there but not being read — same as row 1 |

If row 1 fails, nothing else matters. Fix that first.

## Sanity checks

Run these occasionally:

| Check | Command | Limit |
|---|---|---|
| Index size | `wc -c MEMORY.md` | 16 000 bytes |
| Router size | `wc -c CLAUDE.md` | ~6 000 bytes |
| Biggest project file | `ls -S projects/*/core.md \| head -1` | 16 000 bytes |
| Any `state.md` linked from the index? | `grep state.md MEMORY.md` | should be empty |
| Codex only: is the memory root set? | `grep MEMORY_ROOT ~/.codex/AGENTS.md` | no `{{...}}` left |

## Common mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| Treating `CLAUDE.md` as a manual | It passes 10 KB, replies get generic | Move content to `memory/`, keep the router |
| Appending to `state.md` | It grows every session | Overwrite; push old state to `archived/` |
| Copying logs into project files | Same text in two places, one goes stale | The log is the history; the project file is the present |
| Linking `state.md` from the index | Context cost never drops after splitting | Link `core.md` only |
| Writing rules with no "why" | Rules get ignored or cargo-culted | Every rule records its incident |
