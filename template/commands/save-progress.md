Based on this conversation, save the current project progress and write today's log.

Follow these steps in order.

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
> any project file. Measure it on every run.

1. Measure `MEMORY.md` in bytes.
2. **If > 16 000 bytes → slim it down now** (do not wait until the assistant
   starts giving off-topic answers — that is the late symptom):
   - 🗓️ **Roll the month**: keep only the current month under "Recent Log"; move
     everything older into `logs/archive.md` and leave one pointer line.
   - ✂️ **Cut each entry to one line**: date + 1–2 conclusions + link. The
     play-by-play stays in `logs/YYYY-MM-DD.md`.
3. **If a large new memory file was created**: ask whether it is
   *load-every-time* content or *on-demand* content. On-demand content
   (error banks, reference tables, sample libraries) must live in its own file,
   never inside the index or a always-loaded coach file.

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

🚨 **One-line rule for log entries**: a new day's entry in "Recent Log" is
**one line = date + 1–2 conclusions + link**. The full account goes in
`logs/YYYY-MM-DD.md`. More than one line here is how the index bloats.

---

## Step 5 — Report back

Tell the user:

1. Which memory files were updated (list the filenames)
2. Where the log was written
3. If Step 0 triggered a split: the resulting file structure
4. One closing line: *"Next time you can resume from ___."*

---

## Changelog

- **v2** — Added Step 0 bloat pre-check; changed Step 2 to overwrite-style state
  updates; added the `feedback_memory_no_bloat.md` reference.
  *Root cause*: a project main file ran 49 days and reached 26 K tokens. The
  culprit was v1 of this very command — it appended without archiving and had no
  core/state separation.
- **v2.1** — Added Step 0.5 index health check. *Root cause*: the index itself
  bloats silently, and it is loaded more often than any project file.
