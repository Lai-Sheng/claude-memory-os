# Getting Started

15 minutes to a working setup.

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
> `ls -d ~/.claude/projects/*/`) and gain the native 25 KB / 200-line index cap.
> But that directory is keyed to the folder you **launch Claude Code from** —
> start a session elsewhere and you get a different, possibly empty, store. A
> path you choose is the same everywhere, and easier to back up.

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
3. **`MEMORY.md` → Projects** — list the projects you actually have right now.
   Two is a fine start. **Not in `CLAUDE.md`:** the router's §2 already tells the
   assistant to build its opening menu from this section. Writing the list in both
   files doubles its cost and guarantees they drift apart
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
- [T1 My Project](projects/t1_my_project/core.md) — 🛠️ purpose keywords · key constraint · 🚧 open hook if any
```

Link `core.md` only. Never link `state.md` from the index.

Write the line as **trigger words, not a summary**: the keywords a conversation
about this project would use, plus any open hook (*blocked on X*, *decision
pending*). Explanation belongs in `core.md`. Roughly 120–180 bytes.

## 4. Use it

Start a conversation. The assistant should open with the project menu. Pick one,
work, and if the session produced something worth keeping, before you close the
terminal:

```
/save-progress
```

It prints the index size, warns you if anything is over budget, overwrites
`state.md`, writes today's log, and tells you where to resume.

🚨 **This is not an optional convenience. It is the whole enforcement loop — and
it is a commit, not an autosave.**

Every budget in this architecture is enforced by that command and by nothing
else. It is manual on purpose: a session that was all scratch — a dead end, a
quick look, an experiment you abandoned — *should* evaporate, and not running the
command is how you let it.

What breaks the system is skipping it on a session you meant to keep. Then
`state.md` never gets overwritten, the index never gets rolled, nothing ever moves
to `archived/` — and in about six weeks you arrive at exactly the bloated single
file this repo exists to prevent, with better documentation.

If you will not build that habit, this architecture will not help you, and a
plain `CLAUDE.md` is the honest choice. **Do not fix it with a hook that saves
automatically** — that removes the gate. See
[architecture.md → The commit gate](architecture.md#the-commit-gate-why-save-progress-is-manual).

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
| Did the session **open with the project menu**? | Yes, listing the projects from `MEMORY.md` | The **Projects** section of `MEMORY.md` is still placeholders, or §2 of the router was edited away — check both |
| Does it answer from `state.md`, or improvise? | Quotes your actual resume point | The file is there but not being read — same as row 1 |

If row 1 fails, nothing else matters. Fix that first.

## Sanity checks

Run these occasionally:

| Check | Command | Limit |
|---|---|---|
| Index size | `wc -c MEMORY.md` (also printed by every `/save-progress`) | 16 000 bytes |
| Router size | `wc -c CLAUDE.md` | ~6 000 bytes |
| Biggest project file | `ls -S projects/*/core.md \| head -1` | 16 000 bytes |
| Any `state.md` linked from the index? | `grep state.md MEMORY.md` | should be empty |
| Does the router list projects? | `grep -n "projects/" ~/.claude/CLAUDE.md` | should be empty — the list lives in `MEMORY.md` |
| Longest index line | `awk '{print length}' MEMORY.md \| sort -n \| tail -1` | ~180 characters |

## Common mistakes

| Mistake | Symptom | Fix |
|---|---|---|
| Treating `CLAUDE.md` as a manual | It passes 10 KB, replies get generic | Move content to `memory/`, keep the router |
| Listing projects in `CLAUDE.md` *and* `MEMORY.md` | A new project shows up in one menu but not the other | The list lives in `MEMORY.md` only; the router points at it |
| Index lines that summarise | Lines creep past 250 bytes; the index restates the project files | Keep keywords and open hooks; the explanation is in `core.md` |
| Auto-saving every session | Dead ends and one-off checks pile up as "memory" | `/save-progress` is a commit — run it by hand, on sessions worth keeping |
| Appending to `state.md` | It grows every session | Overwrite; push old state to `archived/` |
| Copying logs into project files | Same text in two places, one goes stale | The log is the history; the project file is the present |
| Linking `state.md` from the index | Context cost never drops after splitting | Link `core.md` only |
| Writing rules with no "why" | Rules get ignored or cargo-culted | Every rule records its incident |
