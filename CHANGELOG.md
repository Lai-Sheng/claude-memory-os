# Changelog

## v0.2.0 — 2026-09-14

v0.1.0 was extracted from a setup that had been running for six months. v0.2.0
is what happened when that setup was audited against the Claude Code it now runs
on — version 2.1.252, not the version it was designed around.

The audit asked three questions: *is this still needed, can it be sharper, and is
any of it redundant?* The answers are below, each with the problem it solves.

### Why an update was needed at all

**Claude Code grew its own memory system after this architecture was built.**
Native memory landed in v2.1.32 (it records and recalls memories as it works)
and was hardened across later releases: `modified` timestamps, a `MEMORY.md`
index capped at 25 KB / 200 lines, a reminder to compact the index as it nears
the cap (v2.1.186), and a hard error instead of silent truncation when it
overflows (v2.1.210).

v0.1.0's README described every layer as something you had to build yourself.
Two of the four are now partly native. Pretending otherwise would have been
inaccurate, and it would have hidden the part that is genuinely still missing.

### Changed

#### 1. The router and the index no longer say the same thing twice

**Problem.** `CLAUDE.md` and `MEMORY.md` are both loaded at the start of every
session, unconditionally, together. In the source setup they each carried the
project list, the frozen-project list, and a large project's zone table — the
same facts, paid for twice on every message. Worse, the old maintenance rule
told you to update *both* whenever a project was created, which guaranteed they
would drift.

**Fix.** A single dividing line:

- **`CLAUDE.md` holds what is true everywhere and rarely changes** — persona,
  opening behaviour, mode gates, hard rules.
- **`MEMORY.md` holds what changes** — the project list, freeze state, the log
  index.
- **The same fact may live in exactly one of them.** The router's opening menu
  now *reads* the project list from the index instead of containing one.

**Result in the source setup:** the always-loaded layer went from **22,468 to
14,870 bytes (−34%)** with no loss of information — every removed rule was
confirmed, by search, to exist in the file it was moved to before it was cut.

#### 2. Index lines are trigger words, not summaries

**Problem.** The rule was "one line per file". The lines obeyed it and grew
anyway: project entries in the source setup averaged **278 bytes**, and a
spot-check of their details found **every one already present** in the file the
line linked to. The index was a second, abbreviated copy of the project files.

**Fix.** An index line keeps **keywords and open hooks** — *"blocked on X"*,
*"decision pending"*, *"results not in yet"* — and drops explanation. Roughly
120–180 bytes.

The hooks are the part that must survive. A line trimmed down to a bare filename
still works when you name the project, but fails when a conversation only
brushes against it without naming it. The hook is what lets the assistant
recognise an unfinished thread it was not explicitly pointed at.

**Result:** the project section went from 8,809 to 4,400 bytes.

#### 3. `/save-progress` reports the index size every run, not only when it is over

**Problem.** Step 0.5 said: *measure the index; if it is over 16 KB, slim it
down.* In the source setup the index reached **16,003 bytes** — over the line —
and nobody noticed. A check that only speaks when a threshold is crossed is
only as reliable as the person remembering to run the check properly.

**Fix.** Step 0.5 now prints one line, unconditionally, every run:
`📐 index: X bytes / Y lines`. You watch the trend instead of waiting for an
alarm. Crossing the budget still produces a proposal to slim down.

If your memory root is Claude Code's own memory directory, the native cap
(25 KB / 200 lines) is a second backstop. If you chose your own path, the
native cap does not apply — the printed number is your only gauge.

#### 4. The commit gate is now stated, not implied

**Problem.** Nothing in v0.1.0 said *why* `/save-progress` is manual. The
obvious-looking improvement — trigger it automatically on session end — was
proposed during the audit, and it would have destroyed the system's main
property.

**Fix.** The docs now say it plainly. `/save-progress` is a **commit**, not a
save. A session is a working tree; most of what happens in it is scratch that
should evaporate. Native memory defaults to *remember what looks useful*; this
architecture defaults to *discard, and keep what you deliberately commit*.
Sessions you choose not to save are the design working, not the habit failing.

#### 5. One memory root, on purpose

**Problem.** The docs never explained why everything lives in one tree, so the
single store read like an accident waiting to be "fixed" with per-folder stores.

**Fix.** `docs/architecture.md` now explains the trade: one root is what lets the
assistant open every session with a menu of *all* your projects. Per-folder
stores give isolation for free and lose the menu. The layering (core/state,
zoning, freezing) is the price of keeping it — paid deliberately.

The install notes also warn about a real trap: Claude Code's native memory
directory is keyed to the **working directory**. Point your memory root there,
start a session from a different folder, and you get a different — possibly
empty — store.

### Removed

- **Codex support** — `template/AGENTS.md`, `docs/codex.md`, and the Codex install
  paths. Running this architecture on Codex is documented in a separate repo;
  keeping a second, drifting copy here contradicted rule 1 above.

### Verified for this release

- Install commands re-run from a fresh clone against an isolated `HOME` on
  Windows 11 · Git Bash and Windows PowerShell 5.1.
- No internal links broken in `examples/`; no placeholders left in it.
- See [README → Status & testing](README.md#status--testing) for what is *not*
  verified.

---

## v0.1.0 — 2026-09-06

First release: four-layer architecture (router · index · core/state · logs),
the `/save-progress` enforcement loop, feedback files, SRP zoning, a fully
filled-in example, and install instructions tested on Windows.
