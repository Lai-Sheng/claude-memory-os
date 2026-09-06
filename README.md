# claude-memory-os

**A file-based memory and project operating system for [Claude Code](https://claude.com/claude-code).**

Claude Code gives you a `CLAUDE.md` and a memory directory. It does not tell you
how to organise them — so most setups end the same way: one file per project,
appended to forever, until every conversation loads twenty thousand tokens of
bugs you fixed in March.

This repo is the structure that fixes it, extracted from a setup that has been
running daily for six months across a dozen concurrent projects.

[中文說明 → README.zh-TW.md](README.zh-TW.md)

---

## The one idea

> **Separate what must always be loaded from what should be loaded on demand —
> and make a routine enforce the boundary.**

Everything else follows from that.

```
┌──────────────────────────────────────────────────────────────┐
│ L1  CLAUDE.md            ~6 KB    ALWAYS loaded              │
│     persona · router menu · mode gates · hard rules          │
├──────────────────────────────────────────────────────────────┤
│ L2  MEMORY.md            <16 KB   ALWAYS loaded              │
│     one line per memory file · recent log · project index    │
├──────────────────────────────────────────────────────────────┤
│ L3  projects/{p}/core.md          WHEN PROJECT PICKED        │
│     why it exists · inviolable rules · design constitution   │
│     projects/{p}/state.md         ON DEMAND (overwritten)    │
│     in-flight work · blockers · resume point                 │
├──────────────────────────────────────────────────────────────┤
│ L4  logs/YYYY-MM-DD.md            ONLY WHEN NAMED            │
│     projects/{p}/archived/*.md    chronological + cold state │
└──────────────────────────────────────────────────────────────┘
```

Each layer has a **byte budget** and a **loading trigger**. Break the budget, get
split. Get loaded more often than your trigger justifies, get demoted.

---

## What you actually get

### 1. A router instead of a manual

`CLAUDE.md` answers three questions and nothing else: who am I talking to, what
can we work on, and what is never negotiable.

The router menu is the part people copy first. At the start of every
conversation the assistant lists your active projects and you just point.
Recognition, not recall — which means project #12 is as reachable as project #1.

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

### 3. `/save-progress` — the enforcement loop

Architecture that relies on discipline decays. This one is enforced by a routine
you run before closing the terminal:

- **Step 0** — measures the target file *before* writing. Over budget → proposes
  a split, and lets you choose whether to do it now
- **Step 0.5** — measures the index *every run*. Over 16 KB → rolls the month
  into the archive, cuts entries to one line
- **Steps 1–5** — overwrites `state.md`, writes the day log, updates the index,
  and tells you exactly where to resume

Step 0 exists because **the first version of this command caused the bloat it now
prevents.** An append-only save routine adds ~300 bytes per session: invisible
for two months, then it owns half your context window.

### 4. Feedback files — corrections that outlive the conversation

When you correct *how* the assistant works, that becomes a small always-loaded
file with a rule, the incident behind it, and how to apply it. The **why** is not
decoration: a rule with a recorded reason can be re-evaluated later; a rule
without one becomes cargo cult.

### 5. Zoning, for projects that outgrow one file

When a project is really several jobs wearing one name, split it into zones with
one responsibility each — under three rules: **facts sink** to a shared
directory, **state belongs to whoever drives it**, and **cross-zone references
are links, never copies**.

See [docs/srp-zoning.md](docs/srp-zoning.md).

---

## Install

```bash
git clone https://github.com/Lai-Sheng/claude-memory-os.git
cd claude-memory-os

cp template/CLAUDE.md      ~/.claude/CLAUDE.md
cp template/commands/*.md  ~/.claude/commands/
cp -r template/memory/*    ~/.claude/projects/<workspace>/memory/
```

Then fill in the `{{PLACEHOLDERS}}`, create your first project from
`template/memory/projects/_project_template/`, and run `/save-progress` at the
end of your next session.

Full walkthrough: **[docs/getting-started.md](docs/getting-started.md)**

⚠️ Already have a `CLAUDE.md`? **Merge**, do not overwrite.

---

## Docs

| Doc | What's in it |
|---|---|
| [getting-started.md](docs/getting-started.md) | 15-minute setup, sanity checks, common mistakes |
| [architecture.md](docs/architecture.md) | The four layers and why each boundary exists |
| [srp-zoning.md](docs/srp-zoning.md) | Splitting a project that outgrew core/state |

---

## Repo layout

```
template/
├── CLAUDE.md                          L1 router
├── commands/
│   ├── save-progress.md               the enforcement loop
│   └── commands.md
└── memory/
    ├── MEMORY.md                      L2 index
    ├── user_profile.md
    ├── feedback_memory_no_bloat.md    the governance rule
    ├── projects/_project_template/    core.md · state.md · archived/
    └── logs/                          day files + archive.md
docs/
```

---

## Design principles

1. **Loading is a cost.** Every always-loaded byte is paid on every message.
2. **Budgets, not intentions.** A limit nobody measures is a wish.
3. **Overwrite beats append.** Append-only structures have no natural size.
4. **History and state are different files.** Logs answer *when*; project files
   answer *now*. Never both.
5. **Recognition beats recall.** Offer the menu; never make the user remember it.
6. **Rules carry their reasons.** A rule without a recorded why gets cargo-culted
   or ignored.
7. **Freezing beats deleting.** A finished project should stop being loaded, not
   stop existing.

---

## Is this for you?

**Probably yes** if you run several long-lived projects through Claude Code and
have noticed replies drifting toward last month's topic.

**Probably not** if you use Claude Code for one repo at a time and start fresh
each session — plain `CLAUDE.md` is fine, and this would be overhead.

**Not Claude-specific in principle.** The layering applies to any agent that
loads files into a context window; only the paths and the slash-command format
are Claude Code specifics.

---

## License

MIT — see [LICENSE](LICENSE). Take it, fork it, rename it. If it saves you a
context window, that's the whole point.
