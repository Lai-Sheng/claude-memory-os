# Running this on Codex

The architecture is markdown discipline, not a Claude feature. It ports to
[Codex](https://developers.openai.com/codex) with **one real change** and a
handful of path swaps.

## The mapping

| Layer | Claude Code | Codex |
|---|---|---|
| L1 router | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` |
| Slash commands | `~/.claude/commands/*.md` | `~/.codex/commands/*.md` |
| Skills | `~/.claude/skills/` | `~/.codex/skills/` |
| Config | `settings.json` | `config.toml` |
| L2–L4 memory | `~/.claude/projects/<ws>/memory/` | **you choose** — e.g. `~/AgentMemory/` |

L2, L3 and L4 are plain markdown in a directory. They are **identical** on both
platforms — the same `MEMORY.md`, the same `core.md` / `state.md` split, the same
`logs/`. Only the router and the command directory differ.

## The one real difference

> **Codex auto-loads `AGENTS.md` and nothing else.**

The index layer has to be **fetched by instruction**, which is what these two do:

1. **§0 Memory Bootstrap in `AGENTS.md`** — declares the memory root and orders
   the agent to read `MEMORY.md` before answering any project question.
2. **`memory/README.md`** — declares that directory the single source of truth,
   so the ownership survives outside the agent's context.

**Do not delete §0.** Without it the agent answers from the router menu alone and
quietly skips the memory layer — a failure that looks fine right up until it
contradicts a decision you recorded last week. It costs one tool call per
session, which is far cheaper than the context the layering saves.

> `CLAUDE.md` ships with the same §0. Claude Code may pull the index in on its
> own depending on version and configuration, but relying on that is a bet — and
> declaring the root explicitly is what lets you keep memory at a path you chose
> instead of an auto-derived one. Same section, same reason, both platforms.

## Install

```bash
git clone https://github.com/Lai-Sheng/claude-memory-os.git
cd claude-memory-os
```

```bash
mkdir -p ~/AgentMemory ~/.codex/commands
cp template/AGENTS.md     ~/.codex/AGENTS.md
cp template/commands/*.md ~/.codex/commands/
cp -r template/memory/.   ~/AgentMemory/
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force ~\AgentMemory, ~\.codex\commands | Out-Null
Copy-Item template\AGENTS.md ~\.codex\AGENTS.md
Copy-Item template\commands\*.md ~\.codex\commands\
Copy-Item -Recurse -Force template\memory\* ~\AgentMemory\
```

Then:

1. Set `{{MEMORY_ROOT}}` in **both** `~/.codex/AGENTS.md` §0 and
   `~/AgentMemory/README.md` — an absolute path, spelled identically in both
2. Fill in the remaining `{{PLACEHOLDERS}}` (persona, router, hard rules)
3. Create your first project from `template/memory/projects/_project_template/`
4. Run `/save-progress` at the end of your next session

⚠️ Already have an `AGENTS.md`? **Merge**, do not overwrite.

## AGENTS.md scoping

Codex reads a global `AGENTS.md` from `~/.codex/`, and project-level `AGENTS.md`
files from the directory you are working in — merged, most specific last. Check
your Codex version's docs for exact precedence before relying on it.

**Put the router in the global file.** Project-level `AGENTS.md` files are for
build commands, test invocations and repo conventions — the things a contributor
would need. The memory architecture is about *you*, not about the repo, and
duplicating §0 into every project is how the two copies drift apart.

## Verify it works

Start a session and ask something only the memory layer knows — the state of a
project you have not mentioned. Then check:

- Did it **read** `MEMORY.md`, or answer from the router menu? (The tool call is
  visible in the transcript. If there is no read, §0 is not landing — check the
  path and that you edited `~/.codex/AGENTS.md`, not a project-level one.)
- Did it open **only** the files that project points to, or the whole tree?
- Did it open `logs/` without being given a date? It should not have.

## Running Codex and Claude Code side by side

This works well, and it is worth doing deliberately rather than by accident.

**Give each agent its own memory root.** Two agents writing the same tree
overwrite each other's `state.md` silently — there is no merge, just a loser.

```
~/AgentMemory/          ← owned by Codex        (declared in ~/.codex/AGENTS.md)
~/.claude/.../memory/   ← owned by Claude Code  (declared in ~/.claude/CLAUDE.md)
```

Then declare the boundary in each `memory/README.md`:

- **this root is owned by** {{agent}}
- **the other root is read-only fallback** — opened only for explicitly requested
  history, missing context, or a migration/audit task
- **never write to the other root** unless the user asks

**Split by project, not by task.** "Codex owns the trading projects, Claude owns
the application projects" works. "Whichever one is open" does not — the state
ends up half in each, and neither is trustworthy.

**When a project moves**, write a handoff file that points at the new source of
truth, and mark the project 📦 in the old index instead of deleting it. The old
agent should then answer *"that moved to X"* rather than working from a stale
copy — which is exactly what the 📦 lifecycle marker is for.
