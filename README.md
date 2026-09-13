# claude-memory-os

**A file-based memory and project operating system for
[Claude Code](https://claude.com/claude-code).**

Claude Code gives you a `CLAUDE.md` and a memory system. It does not tell you how
to organise them — so most setups end the same way: one file per project,
appended to forever, until every conversation loads twenty thousand tokens of
bugs you fixed in March.

This repo is the structure that fixes it, extracted from a setup that has been
running daily for six months across a dozen concurrent projects.

> **v0.2.0 (2026-09-14)** — re-audited against Claude Code 2.1.252. The router and
> the index no longer duplicate each other (−34% always-loaded bytes in the source
> setup), index lines became trigger words, the save routine reports the index
> size on every run, and Codex support moved to its own repo.
> **[What changed and why →](CHANGELOG.md)**

[中文說明 → README.zh-TW.md](README.zh-TW.md)

---

## The one idea

> **Separate what must always be loaded from what should be loaded on demand —
> and make a routine enforce the boundary.**

Everything else follows from that.

![The four layers, each with a byte budget and a loading trigger](docs/assets/layers.svg)

Each layer has a **byte budget** and a **loading trigger**. Break the budget, get
split. Get loaded more often than your trigger justifies, get demoted.

---

## What Claude Code does natively — and what this adds

Claude Code has had its own memory system since v2.1.32: it records and recalls
memories as it works, keeps a `MEMORY.md` index, stamps files with `modified`
timestamps, and since v2.1.186 reminds the agent to compact that index as it
approaches the limit — v2.1.210 turned an overflowing index into a hard error
rather than silent truncation (25 KB / 200 lines).

So if you are weighing that against this, here is the honest split:

| | Native memory | This repo |
|---|---|---|
| Records and recalls memories | ✅ | builds on it |
| `MEMORY.md` index, size-enforced | ✅ | builds on it |
| Always-loaded vs on-demand split | ❌ | **L3 `core` / `state`** |
| Time-series record of what happened when | ❌ | **L4 `logs/`** |
| One project split by responsibility | ❌ | **SRP zoning** |
| Freeze and thaw a finished project | ❌ | **freeze state** |
| **Who decides what gets remembered** | the agent | **you** |

That last row is the one that matters.

### The commit gate

Native memory's default is *remember what looks useful*. This repo's default is
the opposite: **discard everything, and keep only what you deliberately commit.**

`/save-progress` is not a save routine. It is a `commit`.

A session is a working tree. Most of what happens in one is scratch — the branch
you abandoned, the approach that did not work, the file you opened once to check
something. That material is not memory that got lost. It is memory that was never
supposed to exist, and it should evaporate when the session ends.

This is also why the command is manual, and stays manual. Same reason `git commit`
is manual: *what belongs in the history* is a human judgement, and a routine that
fires on its own cannot make it. Automating `/save-progress` would not improve this
system — it would remove the property the whole thing is built around.

If you would rather the agent remember more on its own, you want native memory,
and you should just use it. If you would rather decide, you want a gate.

---

## What you actually get

### 1. A router that never repeats the index

`CLAUDE.md` holds only what is **true everywhere and rarely changes**: who am I
talking to, how the session opens, which capabilities are gated, and what is
never negotiable.

At the start of every conversation the assistant offers a menu of your active
projects and you just point. Recognition, not recall — project #12 is as
reachable as project #1. But the menu's contents live in **`MEMORY.md`**, not in
the router. Both files are loaded together every session; a fact written in both
is paid for twice and eventually disagrees with itself.

> **Rule: the same fact lives in exactly one of the two files.**

### 2. core / state / archived

| | `core.md` | `state.md` | `archived/` |
|---|---|---|---|
| **Holds** | Why it exists, inviolable rules, design constitution | Active work, blockers, resume point | Historical snapshots |
| **Write mode** | Rarely, on strategic pivot | **Overwritten each phase** | New dated files |
| **Loaded** | When the project is picked | On demand | Only when named |
| **Test** | "True in a year?" | "Matters in 30 days?" | "Explains something?" |

The write mode is the load-bearing part. `state.md` is **overwritten, never
appended**. Old state either moves to `archived/` or gets deleted — the log
already recorded it, and storing it twice is the bug, not the safety net.

### 3. An index of trigger words

`MEMORY.md` is one line per file, and each line is a **trigger, not a summary**:
the keywords that let the assistant recognise the topic, plus any **open hook** —
*blocked on X*, *decision pending*, *results not in yet*. No explanation; that is
what the linked file is for. Roughly 120–180 bytes a line.

Keep the hooks even when you cut everything else. A bare filename still works
when you name the project; the hook is what lets the assistant notice an
unfinished thread when a conversation only brushes against it.

### 4. `/save-progress` — the enforcement loop

Architecture that relies on discipline decays. This one is enforced by a routine
you run whenever a session produced something worth keeping:

- **Step 0** — measures the target file *before* writing. Over budget → proposes
  a split, and lets you choose whether to do it now
- **Step 0.5** — prints the index size **on every run**, not only when it is over.
  Over budget → proposes rolling the month and trimming lines
- **Steps 1–5** — overwrites `state.md`, writes the day log, updates the index,
  and tells you exactly where to resume

Step 0 exists because **the first version of this command caused the bloat it now
prevents.** Step 0.5 prints unconditionally because the conditional version let
an index sit at 16,003 bytes — over a 16,000-byte limit — without anyone noticing.

### 5. Feedback files — corrections that outlive the conversation

When you correct *how* the assistant works, that becomes a small always-loaded
file with a rule, the incident behind it, and how to apply it. The **why** is not
decoration: a rule with a recorded reason can be re-evaluated later; a rule
without one becomes cargo cult.

### 6. Zoning, for projects that outgrow one file

When a project is really several jobs wearing one name, split it into zones with
one responsibility each — under three rules: **facts sink** to a shared
directory, **state belongs to whoever drives it**, and **cross-zone references
are links, never copies**.

See [docs/srp-zoning.md](docs/srp-zoning.md).

---

## Before you install: the habit this depends on

Every budget here is enforced by **one command**:

```
/save-progress
```

Run it at the end of any session that produced something you want to keep.
Sessions you *choose* not to save are the design working — that is the commit
gate. What breaks the system is skipping it on sessions you did mean to keep:
`state.md` is never overwritten, the index is never rolled, nothing ever moves
to `archived/`, and in about six weeks you arrive at exactly the bloated single
file this repo exists to prevent — only with better documentation explaining
what you should have been doing.

The structure is not the system. The structure plus that habit is the system. If
you know you will not run it, a plain `CLAUDE.md` is the honest choice and this
would only be overhead.

## See it filled in first

`template/` ships with placeholders, which makes it hard to picture. The
**[examples/](examples/)** directory is the same architecture fully filled in for
a fictional developer three months in — a router that points at its index, an
index of trigger-word lines, a `core.md` with no dates in it, a `state.md` with a
real resume point, an archived decision record, and a feedback file that cites
the incident behind its rule.

Read that first. Copy from `template/`.

## Install

```bash
git clone https://github.com/Lai-Sheng/claude-memory-os.git
cd claude-memory-os
```

### Step 1 — copy the files

Memory can live **anywhere you like** — you declare the path in the router file,
so there is no magic directory to hunt for. These commands use `~/AgentMemory`.

**macOS / Linux**

```bash
mkdir -p ~/AgentMemory ~/.claude/commands
cp template/CLAUDE.md     ~/.claude/CLAUDE.md
cp template/commands/*.md ~/.claude/commands/
cp -r template/memory/.   ~/AgentMemory/
```

**Windows PowerShell**

```powershell
New-Item -ItemType Directory -Force ~\AgentMemory, ~\.claude\commands | Out-Null
Copy-Item template\CLAUDE.md ~\.claude\CLAUDE.md
Copy-Item template\commands\*.md ~\.claude\commands\
Copy-Item -Recurse -Force template\memory\* ~\AgentMemory\
```

⚠️ Already have a `CLAUDE.md`? **Merge**, do not overwrite.

### Step 2 — point the router at your memory 🚨

Open `~/.claude/CLAUDE.md` and replace `{{MEMORY_ROOT}}` in **§0** with the
absolute path you used. Do the same in `~/AgentMemory/README.md`.

**§0 is what makes the memory layer load. Do not delete it.** Without it the
agent answers from the router alone and silently skips memory — which looks fine
until it contradicts something you recorded last week.

### Step 3 — fill in the rest, then check

Fill in the persona in `CLAUDE.md`, your projects under **Projects** in
`MEMORY.md` (the router reads its menu from there), and `user_profile.md`. Then
verify nothing is left blank:

```bash
grep -rn "{{" ~/.claude/CLAUDE.md ~/AgentMemory/
```

```powershell
Get-ChildItem -Recurse -File ~\.claude\CLAUDE.md, ~\AgentMemory | Select-String -SimpleMatch "{{"
```

Every line it prints is a placeholder you still owe. When it prints nothing,
create your first project from
`template/memory/projects/_project_template/` and run `/save-progress` at the end
of your next session worth keeping.

Full walkthrough: **[docs/getting-started.md](docs/getting-started.md)**

> **Prefer Claude Code's own memory directory?** You can point `{{MEMORY_ROOT}}`
> at `~/.claude/projects/<workspace>/memory/` — find yours with
> `ls -d ~/.claude/projects/*/`. You gain the native 25 KB / 200-line backstop.
> But that directory is **keyed to the folder you start Claude Code from**: start
> a session somewhere else and you get a different, possibly empty, store. A path
> you choose yourself is the same everywhere, and easier to back up.

---

## Docs

| Doc | What's in it |
|---|---|
| [getting-started.md](docs/getting-started.md) | 15-minute setup, verification, common mistakes |
| [architecture.md](docs/architecture.md) | The four layers, why each boundary exists, why one memory root |
| [srp-zoning.md](docs/srp-zoning.md) | Splitting a project that outgrew core/state |
| [CHANGELOG.md](CHANGELOG.md) | What changed in each release, and the problem each change solves |
| [examples/](examples/) | The whole thing filled in for a fictional user — read this first |

---

## Repo layout

```
examples/                              the same architecture, filled in
template/
├── CLAUDE.md                          L1 router — global, stable, no project list
├── commands/
│   ├── save-progress.md               the enforcement loop (the commit)
│   └── commands.md
└── memory/                            plain markdown
    ├── README.md                      memory-root ownership declaration
    ├── MEMORY.md                      L2 index — the menu's only source
    ├── user_profile.md
    ├── feedback_memory_no_bloat.md    the governance rule
    ├── projects/_project_template/    core.md · state.md · archived/
    └── logs/                          day files + archive.md
extras/                                unrelated bonus commands, kept apart on purpose
docs/
CHANGELOG.md
```

---

## Design principles

1. **Loading is a cost.** Every always-loaded byte is paid on every message.
2. **Budgets, not intentions.** A limit nobody measures is a wish.
3. **Show the number, don't wait for the alarm.** A check that only speaks past a
   threshold fails silently the day nobody runs it carefully.
4. **Overwrite beats append.** Append-only structures have no natural size.
5. **History and state are different files.** Logs answer *when*; project files
   answer *now*. Never both.
6. **Say it once.** A fact in two always-loaded files is paid twice and drifts.
7. **Recognition beats recall.** Offer the menu; never make the user remember it.
8. **Discard by default.** Keep what you commit; let the rest evaporate.
9. **Rules carry their reasons.** A rule without a recorded why gets cargo-culted
   or ignored.
10. **Freezing beats deleting.** A finished project should stop being loaded, not
    stop existing.

---

## Is this for you?

**Probably yes** if you run several long-lived projects through Claude Code and
have noticed replies drifting toward last month's topic.

**Probably not** if you use Claude Code for one repo at a time and start fresh
each session — plain `CLAUDE.md` is fine, and this would be overhead.

**Also probably not** if you would rather the agent decide what to remember. That
is what native memory is for, and it does it well.

---

## Extras

`extras/` holds standalone Claude Code commands that have nothing to do with
the memory architecture above — kept separate on purpose, so the one idea this
repo is about doesn't get diluted. **[extras/README.md →](extras/README.md)**

---

## Status & testing

Being explicit about what has actually been verified, because "it worked on my
machine" is how the first version of these install instructions shipped broken.

**Verified — 2026-09-14 (v0.2.0)**

- Install commands run end to end from a fresh clone against an isolated `HOME`:
  **Windows 11 · Git Bash** and **Windows PowerShell 5.1**, exit 0, every
  template file landed.
- The placeholder self-check reports correctly.
- No internal link in `examples/` is broken; no placeholders remain in it.

**Not verified**

- **macOS and Linux natively.** The shell commands are plain POSIX (`mkdir -p`,
  `cp -r`) and should work, but nobody has run them there. If you do, an issue
  either way is welcome.
- **Agent behaviour.** Whether a given Claude Code build honours §0, and reads the
  menu from `MEMORY.md` as instructed, has not been tested across versions —
  which is exactly why
  [the verification steps](docs/getting-started.md#verify-it-actually-works)
  exist. Run them after installing; do not assume.

**Provenance**

The architecture is not theoretical — it was extracted from a setup that has run
daily since March 2026 across roughly a dozen concurrent projects, and the rules
in it are the ones that survived contact with that. v0.2.0's numbers come from
auditing that same setup. The *packaging* is where bugs will be. Please report
them.

## License

MIT — see [LICENSE](LICENSE). Take it, fork it, rename it. If it saves you a
context window, that's the whole point.
