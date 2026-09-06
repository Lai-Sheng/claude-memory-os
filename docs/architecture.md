# Architecture

## The problem this solves

Claude Code gives you `CLAUDE.md` and a memory directory. Nothing tells you how
to *organise* them. So the default outcome is:

1. `CLAUDE.md` slowly becomes a dumping ground of instructions
2. One memory file per project, appended to forever
3. Six weeks later, every conversation loads 26 K tokens of solved bugs
4. The assistant starts answering questions you did not ask, because the
   loudest thing in its context is a topic from three weeks ago

The failure is not that memory is *wrong*. It is that **everything is loaded all
the time**, so relevance decays into noise.

This architecture fixes that with one idea:

> **Separate what must always be loaded from what should be loaded on demand,
> and make a routine enforce the boundary.**

---

## The four layers

```
┌─────────────────────────────────────────────────────────────┐
│ L1  CLAUDE.md            ~6 KB   ALWAYS loaded              │
│     persona · router menu · mode gates · hard rules         │
├─────────────────────────────────────────────────────────────┤
│ L2  MEMORY.md            <16 KB  ALWAYS loaded              │
│     one line per memory file · recent log · project index   │
├─────────────────────────────────────────────────────────────┤
│ L3  projects/{p}/core.md         loaded WHEN PROJECT PICKED │
│     why it exists · inviolable rules · design constitution  │
│                                                             │
│     projects/{p}/state.md        loaded ON DEMAND           │
│     in-flight work · blockers · resume point (OVERWRITTEN)  │
├─────────────────────────────────────────────────────────────┤
│ L4  logs/YYYY-MM-DD.md           loaded ONLY WHEN NAMED     │
│     projects/{p}/archived/*.md   chronological + cold state │
└─────────────────────────────────────────────────────────────┘
```

Each layer has a **byte budget** and a **loading trigger**. A file that violates
its budget gets split; a file that gets loaded more often than its trigger
justifies gets demoted.

---

## Layer 1 — `CLAUDE.md` is a router, not a manual

The single most common mistake is treating `CLAUDE.md` as the place where all
instructions go. It is not. It answers exactly three questions:

| Question | Section |
|---|---|
| Who am I talking to, and how? | Persona |
| What can we work on? | Router menu |
| What is never negotiable? | Hard rules + mode gates |

**The router menu is the key insight.** The user should never have to remember
what projects exist. At the start of every conversation the assistant offers the
menu; the user just points. This converts recall into recognition — and it means
project #12 is exactly as reachable as project #1.

**Mode gates** matter because capabilities collide. A skill that creates files is
great while coding and actively harmful during a planning conversation. Declaring
"these skills activate only in development contexts" prevents the assistant from
helpfully building something nobody asked for.

---

## Layer 2 — `MEMORY.md` is an index, never a container

This is the file that bloats most invisibly, because it is loaded *more often
than anything else*. The rule is absolute:

> **One line per memory file. If it needs two lines, the content belongs in the
> file it points to.**

The index links `core.md` and never `state.md` — because state is read on demand,
and linking it from the index would defeat the entire split.

The "Recent Log" section carries **one line per day**: date, one or two
conclusions, link. The play-by-play is in the day file. At the start of each
month, the previous month rolls into `logs/archive.md`.

---

## Layer 3 — core / state / archived

This is the load-bearing split.

| | `core.md` | `state.md` | `archived/` |
|---|---|---|---|
| **Contains** | Why it exists, inviolable rules, design constitution | Active work, blockers, resume point | Historical state snapshots |
| **Write mode** | Rarely, on strategic pivot | **Overwrite each phase** | Append new dated files |
| **Loaded** | When the project is picked | On demand | Only when named |
| **Test** | "True in a year?" | "Matters in 30 days?" | "Was true, still explains something" |

The write-mode difference is what actually prevents bloat. `state.md` is
**overwritten**, not appended. New state comes in; old state either moves to
`archived/` (if it still explains something) or is deleted (the log already
recorded it — keeping it twice is the bug, not the safety net).

### The 30-day test

Before writing a line into `state.md`, ask: *will this still matter in 30 days?*

- **Yes** → `state.md`, or promote it to `core.md`
- **No** → it belongs in `logs/` and nowhere else

Most content fails this test. That is the point.

---

## Layer 4 — logs vs project files

These two are constantly confused, and conflating them is the most common cause
of double storage.

| | Logs | Project files |
|---|---|---|
| **Indexed by** | Date | Project |
| **Answers** | "What happened on the 28th?" | "Where does X stand now?" |
| **Grows** | One file per day, forever | Bounded, overwritten |

**Never copy a log entry into a project file.** The log is already the source of
truth for history. The project file keeps only what is *still true*.

---

## The enforcement loop

Architecture that depends on discipline decays. This one is enforced by a
routine — `/save-progress` — that runs at the end of a session and:

1. **Measures the target file before writing** (Step 0). Over budget → propose
   a split, with the user choosing whether to do it now
2. **Measures the index every single time** (Step 0.5). Over 16 KB → roll the
   month, cut entries to one line
3. **Overwrites `state.md`** instead of appending
4. **Writes the day log** separately, with the resume point
5. **Reports** what changed and where to pick up

Step 0 exists because **the previous version of this very command caused the
bloat it now prevents**. An append-only save routine adds ~300 bytes a session:
invisible for two months, then it owns half your context window.

---

## Feedback files: turning corrections into rules

When the user corrects how you work — not what you built, but *how* — that
correction should outlive the conversation. Feedback files do this:

```markdown
---
name: feedback-answer-actual-question
description: Replies must grow from the literal request in the current message
metadata:
  type: feedback
---

The rule.

**Why:** the incident that produced it.
**How to apply:** what to do differently, concretely.
```

The **Why** line is not decoration. A rule whose reason is recorded can be
re-evaluated when circumstances change; a rule without one becomes cargo cult and
eventually gets ignored.

Feedback files are indexed in `MEMORY.md` and always loaded — they are small, and
they are the difference between an assistant that learns and one that repeats
itself.

---

## Numbering and identity

Category letter + serial: `T1`, `A1`, `K6`, `H1`.

- **Retired numbers are never reused** — so a log entry from March is still
  unambiguous in September
- **Filenames never change** after creation, even when a project is renamed —
  the index carries the display name, the filesystem carries identity

This costs nothing and saves you from the day you rename something and break
forty cross-references.

---

## Lifecycle

| Marker | Meaning | Loading behaviour |
|---|---|---|
| 🔥 | Hard deadline near | Report status first when opened |
| ✅ | Active | Normal |
| 🚧 | Blocked / evaluating | State the blocker when opened |
| ❄️ | Frozen | **Never auto-load.** Files kept; thaw on request |
| 📦 | Handed off | Point at the handoff doc; do not work on it here |

Freezing is the underrated one. A finished project's memory does not need
deleting — it needs to stop being loaded. Mark it ❄️, drop it from the router
menu, keep the files. If the user ever asks, thaw it.
