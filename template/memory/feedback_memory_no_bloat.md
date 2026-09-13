---
name: feedback-memory-no-bloat
description: 🌟 Global memory governance — core/state split, no append-growth, auto-split on trigger; always-loaded layer says each fact once, index lines are trigger words
metadata:
  type: feedback
  tier: core
---

# Global Memory Governance: The No-Bloat Rule

A project main file ran for 49 days and reached 26 K tokens. Every conversation
about that project was paying for it. The culprit was the `/save-progress`
command: it appended without archiving, and did not separate core from state.

**Why:** stale *state* (fixed bugs, wired-up UI, old instance IDs) and permanent
*core* (specs, constitutions, design patterns) in the same file grow without
bound. Reading it costs the full 26 K every time — meaning every conversation
was blocked by bugs that had been solved months earlier.

**How to apply:** these rules apply **before any write to a project memory file**,
without needing to be reminded.

---

## 🚦 Split triggers (any one → split into core/state)

| # | Condition | Action |
|---|---|---|
| 1 | Main file **> 5 K tokens** (~16 KB / ~200 lines of markdown) | Split into `core.md` + `state.md` |
| 2 | Main file has **5+ date-stamped "completed" sections** | Split, and move the dated sections into `archived/` |
| 3 | Main file holds **perishable snapshots** — instance IDs, file mtimes, pixel sizes, version numbers | That section moves to `state.md` or `archived/` |
| 4 | Main file holds a **"lesson learned" section that has already been promoted to a feedback file** | Delete the section — the feedback file is the single source of truth |

**Exception**: pure design documents (a script, a mechanics list, a spec) are
*core by nature* and do not bloat. They are exempt.

---

## 📐 The split layout

```
memory/projects/{p}/
├── core.md              ← permanent: why it exists, inviolable rules, design constitution
├── state.md             ← in-flight work (OVERWRITE, never append; reset each phase)
└── archived/
    └── YYYY-MM_topic.md ← historical state snapshots, read on request only
```

**Index rules in `MEMORY.md`**
- ✅ Link `core.md` — always loaded
- ❌ Never link `state.md` — on demand only
- ❌ Never link `archived/*` — buried

---

## 🎯 What goes where

### `core.md`
- Design constitution, spec, mechanics list, architecture, scope, roles
- **No sentences of the form "completed 2026-04-XX ..."**
- **No mtimes / instance IDs / pixel sizes** — perishable data pollutes core
- Changed only on a strategic pivot

### `state.md`
- Active work, current blockers, last ~30 days of milestones, resume point
- **Overwritten each phase.** New state in, old state to `archived/`
- Ask before writing: *"will this still matter in 30 days?"*
  - Yes → `state.md` (or promote to core)
  - No → it belongs in `logs/` only

### `archived/`
- Historical state that is important but no longer active
- **Filename prefixed by date or topic**: `2026-04_ui_buildout.md`
- One file may cover 1–2 months of themed wrap-up. One file per *day* is the
  job of `logs/`, not this.

---

## 🚫 Never do this

- ❌ Notice a main file is huge but say nothing because the user did not ask —
  **proposing the split is the assistant's job**
- ❌ Copy a full log entry into the project file — double storage; `logs/` is
  already the source of truth
- ❌ Put instance IDs / mtimes / file sizes into `core.md`
- ❌ Copy `state.md` into `archived/` "just to be safe" — one copy is enough;
  history lives in `logs/` and in git

---

## ✅ Worked example

**Before**: one 26 K-token file mixing a mechanics constitution, three months of
UI build notes, instance IDs, a bug list, and design patterns.

**After**:
- `core.md` — mechanics constitution, protagonists, script architecture, design
  patterns (permanent; changes only when the design changes)
- `state.md` — current audio wiring + 16 playtest TODOs + near-term milestones
- `archived/2026-03_to_04_early_buildout.md` — menu UI, font fix, ESC-key saga,
  instance-ID snapshot

**Payoff**: the index links `core.md` (always loaded), `state.md` is read on
demand, `archived/` only when named. ~20 K tokens of context saved per
conversation about that project.

---

## 📐 The always-loaded layer (`CLAUDE.md` + `MEMORY.md`)

Project files are loaded when picked. These two are loaded **every session,
together**, so they get their own rules.

### Say it once

- **`CLAUDE.md`** holds what is true everywhere and rarely changes: persona,
  opening behaviour, mode gates, non-negotiable rules.
- **`MEMORY.md`** holds what changes: the project list, freeze state, the log
  index.
- **The same fact never lives in both.** The router's menu *reads* the project
  list from the index; it does not contain one.

**Why:** in the setup this came from, both files carried the project list, the
frozen list, and a zone table. Enforcing this rule cut the always-loaded layer
from 22,468 to 14,870 bytes (−34%) with nothing lost.

**How to apply:** before removing anything from `CLAUDE.md`, search for it in the
file that should own it. Present → delete. Absent → move it there first.

### Index lines are trigger words

- Keep **keywords** and **open hooks** (*blocked on X*, *decision pending*,
  *results not in yet*). Drop explanation. ~120–180 bytes.
- Never cut the hooks. They are what let the assistant notice an unfinished
  thread when a conversation brushes against a project without naming it.

**Why:** project lines averaged 278 bytes, and every detail in them already
existed in the linked file — the index had become a second, shorter copy.

### Print the number; don't wait for the alarm

`/save-progress` prints the index size on every run. **Why:** the old conditional
check ("if over 16 KB, slim down") let the index reach 16,003 bytes without anyone
noticing.

---

## Related

- The culprit and its rewrite: `commands/save-progress.md`
- Log archiving policy: `feedback_memory_management.md`
