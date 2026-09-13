# Architecture

## The problem this solves

Claude Code gives you `CLAUDE.md` and a memory system. Since v2.1.32 it even
records and recalls memories on its own. What nothing tells you is how to
*organise* any of it. So the default outcome is:

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
│     persona · opening behaviour · mode gates · hard rules   │
│     global and stable — never repeats anything in L2        │
├─────────────────────────────────────────────────────────────┤
│ L2  MEMORY.md            <16 KB  ALWAYS loaded              │
│     project list (the menu's only source) · feedback rules  │
│     · recent log — one trigger-word line per file           │
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

### What is native now, and what is not

Claude Code's own memory covers part of L2: it keeps a `MEMORY.md` index, caps
it at 25 KB / 200 lines, reminds the agent to compact it (v2.1.186) and errors
instead of truncating on overflow (v2.1.210). Those protections apply **only if
your memory root is Claude Code's own memory directory** — a root you chose
yourself gets none of them, which is one more reason the save routine prints the
index size every run.

Nothing native provides L3's core/state split, L4's time-series logs, zoning,
freezing — or the commit gate described at the end of this document.

---

## Layer 1 — `CLAUDE.md` is a router, not a manual

The single most common mistake is treating `CLAUDE.md` as the place where all
instructions go. It is not. It answers exactly three questions:

| Question | Section |
|---|---|
| Who am I talking to, and how? | Persona |
| How does a session open? | Opening behaviour |
| What is never negotiable? | Hard rules + mode gates |

**The opening menu is the key insight.** The user should never have to remember
what projects exist. At the start of every conversation the assistant offers the
menu; the user just points. This converts recall into recognition — and it means
project #12 is exactly as reachable as project #1.

**But the router does not contain the menu. It points at it.** The project list
lives in `MEMORY.md`.

### Why the router must never repeat the index

`CLAUDE.md` and `MEMORY.md` arrive in the same moment, unconditionally, at the
start of every session. There is no "read the router, then decide whether to
read the index". So any fact written in both is:

1. **paid for twice**, on every message of every session, and
2. **maintained twice** — which means that sooner or later only one copy gets
   updated, and the assistant is handed two versions of the truth.

The dividing line:

| `CLAUDE.md` — true everywhere, rarely changes | `MEMORY.md` — changes as work happens |
|---|---|
| Persona and tone | The project list |
| Opening behaviour ("offer the menu from `MEMORY.md`") | Which projects are frozen or handed off |
| Mode gates | Feedback rule index |
| Non-negotiable rules | Recent log |

In the setup this repo was extracted from, the two files each carried the
project list, the frozen list, and a large project's zone table. Enforcing the
line took the always-loaded layer from **22,468 to 14,870 bytes (−34%)** without
losing a single rule — because every rule removed from the router was first
confirmed, by search, to already exist in the file that now owns it.

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

### A line is a trigger, not a summary

"One line" turned out not to be enough of a rule. Lines obey it and still grow:
in the source setup, project lines averaged 278 bytes, and every detail checked in
them already existed in the file they linked to. The index had quietly become an
abbreviated second copy of the project files.

What a line is actually *for* is recognition. So it keeps:

- **keywords** — the words a conversation about this project would use
- **open hooks** — *blocked on X*, *decision pending*, *results not in yet*

and drops everything that explains. Roughly 120–180 bytes.

The hooks are not optional. A line cut to a bare filename still works when the
user names the project. It fails when a conversation only brushes against the
topic without naming it — and the hook is exactly what lets the assistant notice
that an unfinished thread is in play.

**Before cutting a detail, confirm it exists in the target file.** If it does
not, it is not a trim — it is a move, and it goes into the file first.

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

## One memory root, on purpose

Everything lives in one tree — every project, every log. That can look like an
oversight: why not give each project its own folder and its own memory?

Because the opening menu depends on it. The assistant can only offer *all* your
projects if it can see all of them from one index. Per-folder stores give you
isolation for free, and in exchange every session only knows about the folder it
was started in. You would have a dozen assistants that have never heard of each
other's work.

So the trade is explicit:

| One root (this architecture) | Per-folder stores |
|---|---|
| Menu of every project at session start | Only the current folder's project |
| Isolation must be **built** — core/state, zoning, freezing | Isolation is automatic |
| Cross-project links work (`[[other_project]]`) | Cross-project context is invisible |

The layering in L3, zoning, and the freeze state are the price of the left
column. They are paid on purpose.

"All your projects" means all the projects **this assistant is responsible for**.
A project handed to another person or tool is marked 📦 and drops out of the
menu. That is not a hole in the system; it is the boundary working.

> ⚠️ **The native-directory trap.** Claude Code's own memory directory is keyed to
> the working directory you launch from. If you point your memory root at it and
> then start a session from a different folder, you silently get a different —
> possibly empty — store. Choosing your own absolute path, declared in the
> router's §0, avoids this.

---

## The enforcement loop

Architecture that depends on discipline decays. This one is enforced by a
routine — `/save-progress` — that you run at the end of a session worth keeping:

1. **Measures the target file before writing** (Step 0). Over budget → propose
   a split, with the user choosing whether to do it now
2. **Prints the index size, every run** (Step 0.5). Over budget → propose rolling
   the month and trimming lines to trigger words
3. **Overwrites `state.md`** instead of appending
4. **Writes the day log** separately, with the resume point
5. **Reports** what changed and where to pick up

Step 0 exists because **the previous version of this very command caused the
bloat it now prevents**. An append-only save routine adds ~300 bytes a session:
invisible for two months, then it owns half your context window.

Step 0.5 prints unconditionally because the conditional version failed in the
most ordinary way possible. It said *"if the index is over 16 KB, slim it down"*.
The index reached 16,003 bytes and nobody noticed — a check that only speaks past
a threshold is only as reliable as the attention of whoever runs it. A number
printed every time needs no attention; you just watch it move.

---

## The commit gate: why `/save-progress` is manual

The most tempting improvement to this architecture is to trigger `/save-progress`
automatically — on session end, or before context compaction. **Do not.**

`/save-progress` is not a save routine. It is a `commit`.

A session is a working tree. Most of what happens in it is scratch: the approach
that did not work, the branch that was abandoned, the file opened once to check
one thing. That material was never supposed to become memory. The routine being
manual is what lets it evaporate.

This is the one place the architecture deliberately disagrees with native memory:

| | Native memory | This architecture |
|---|---|---|
| Default | Remember what looks useful | Discard |
| What gets kept | Whatever the agent judges useful | What the user commits |
| A session you did not save | Partially remembered anyway | Gone — as intended |

The same reason `git commit` is not automatic applies here: *what belongs in the
history* is a judgement, and a routine that fires on its own cannot make it. The
only failure worth guarding against is forgetting to save a session you *meant*
to keep — and if that happens often enough to matter, the fix is a reminder that
asks, never a hook that writes.

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
| 📦 | Handed off | Out of the menu; point at the handoff note if asked |

Freezing is the underrated one. A finished project's memory does not need
deleting — it needs to stop being loaded. Mark it ❄️, drop it from the menu,
keep the files. If the user ever asks, thaw it.
