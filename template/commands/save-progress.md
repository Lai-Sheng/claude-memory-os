Based on this conversation, save the current project progress and write today's log.

Follow these steps in order.

> 🔒 **This command is a commit, not an autosave.** The user running it is the
> decision that this session is worth keeping. Never suggest triggering it
> automatically, and never write project memory at the end of a session the user
> did not save — unsaved sessions are meant to evaporate.

---

## Step 0 — Bloat pre-check 🚦

**Before writing into any project memory file**, check its condition first.

1. **Measure the target file** (line count or byte count).
2. **If the main file is > 5 K tokens (~16 000 bytes), OR contains 5+
   date-stamped "completed" sections** (e.g. `### Done (2026-04-01)`):
   - ⚠️ Warn the user: *"This project's main file has bloated. I recommend
     splitting it into core/state before writing."*
   - Offer three options and let them pick:
     - **A. Split first** (per `feedback_memory_no_bloat.md`), then append — *recommended*
     - **B. Log only** — write today's log, defer the main file
     - **C. Append anyway** — it keeps growing; the user owns that call
3. **If the project already uses core/state**:
   - New content goes into `state.md` — **overwrite, do not append**
   - Superseded state moves to `archived/YYYY-MM_topic.md`

> **Why this step exists**: an append-only save command is the single most common
> cause of context bloat. A main file that grows 300 bytes per session is
> invisible for two months and then eats half the context window.

---

## Step 0.5 — Index health check 🩺

> The index is loaded **every single time**, so it deserves stricter defence than
> any project file.
>
> 🚨 **Report unconditionally — not only when it is over budget.** A check that
> only speaks past a threshold fails silently: an index once sat at 16,003 bytes,
> over a 16,000-byte limit, and nobody noticed, because nothing said anything.

1. **Every run, print one line, whatever the size:**
   `📐 index: X bytes / Y lines (budget 16 000)`
   The user watches the trend instead of waiting for an alarm.
2. **If over budget → propose a slim-down** (do not wait until the assistant
   starts giving off-topic answers — that is the late symptom):
   - 🗓️ **Roll the month**: keep only the current month under "Recent Log"; move
     everything older into `logs/archive.md` and leave one pointer line.
   - ✂️ **Cut each log entry to one line**: date + 1–2 conclusions + link. The
     play-by-play stays in `logs/YYYY-MM-DD.md`.
   - 🔑 **Turn project lines back into trigger words**: keep keywords and open
     hooks, delete explanation. **Before deleting a detail, confirm it exists in
     the linked file** — if it does not, move it there first.
3. **If a large new memory file was created**: ask whether it is
   *load-every-time* content or *on-demand* content. On-demand content
   (error banks, reference tables, sample libraries) must live in its own file,
   never inside the index or an always-loaded file.

> If the memory root is Claude Code's own memory directory, the native cap
> (25 KB / 200 lines) is a second backstop. With a custom root it does not apply —
> this printed line is the only gauge.

---

## Step 1 — Identify the project

Determine which project(s) this conversation touched.
**Read `MEMORY.md` first** to get the current list of active projects and their
memory paths. Do not rely on recall — the list changes often.

---

## Step 2 — Update the project memory

### Path A: project already uses core/state

- **`state.md` — rewrite it**, do not append. Restate the current active work.
- Old state with historical value → `archived/YYYY-MM_topic.md`
- Old state that is simply obsolete → delete it (the log already recorded it;
  storing it twice is the bug, not the safety net)
- **`core.md` changes only on a strategic pivot** — direction change, spec
  rewrite, constitution amendment

### Path B: project is still a single file (transitional)

- If Step 0 was green (< 5 K tokens): appending is allowed, but evaluate whether
  it is time to promote to core/state
- If Step 0 was red: split first, then write

### What the update must cover

- What was completed this session
- Where it stopped (the exact resume point for next time)
- New bugs / TODOs / decisions
- Any change of architecture or direction

---

## Step 3 — Write today's log

Create or append to `logs/YYYY-MM-DD.md` (today's date).

```markdown
# Log YYYY-MM-DD

## [Project name — zone if applicable]

### Completed
- ...

### Stopped at
- ...

### Next time
- ...

### Notes (optional)
- ...
```

If a log already exists for today, **append a new project section — never
overwrite** what is there.

⚠️ **Division of labour between logs and project files**
- **Log = chronological record.** Answers *"what happened on the 28th?"*
- **Project file = state snapshot.** Answers *"where does project X stand now?"*
- Never copy a full log entry into the project file. That is double storage.
  The project file keeps only what is **still true and still relevant**.

---

## Step 4 — Update the index

If new memory files were created, add their index lines to `MEMORY.md`.

**Index rules**
- ✅ Link `core.md` — always loaded
- ❌ Never link `state.md` — on-demand, saves permanent context
- ❌ Never link `archived/*` — buried, opened only when named
- 🔑 A project line is **trigger words + open hooks**, ~120–180 bytes — not a
  summary of the project file
- 📐 **A new project goes into `MEMORY.md` only.** Never add it to `CLAUDE.md`;
  the router builds its menu from the index

🚨 **One-line rule for log entries**: a new day's entry in "Recent Log" is
**one line = date + 1–2 conclusions + link**. The full account goes in
`logs/YYYY-MM-DD.md`. More than one line here is how the index bloats.

---

## Step 5 — Report back

Tell the user:

1. The Step 0.5 line: `📐 index: X bytes / Y lines`
2. Which memory files were updated (list the filenames)
3. Where the log was written
4. If Step 0 triggered a split: the resulting file structure
5. One closing line: *"Next time you can resume from ___."*

---

## Changelog

- **v2** — Added Step 0 bloat pre-check; changed Step 2 to overwrite-style state
  updates; added the `feedback_memory_no_bloat.md` reference.
  *Root cause*: a project main file ran 49 days and reached 26 K tokens. The
  culprit was v1 of this very command — it appended without archiving and had no
  core/state separation.
- **v2.1** — Added Step 0.5 index health check. *Root cause*: the index itself
  bloats silently, and it is loaded more often than any project file.
- **v2.2** — Step 0.5 prints the index size on every run instead of only past the
  threshold; added trigger-word trimming; new projects go into `MEMORY.md` only;
  stated the commit gate. *Root cause*: the conditional check let an index reach
  16,003 bytes unnoticed, and project lines had grown into summaries duplicating
  their own files.
