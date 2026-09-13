# Memory Index

<!-- FILLED-IN EXAMPLE. Well under the 16 KB budget. -->

> ⚠️ Loaded every conversation. An *index*, never a container. One line per file.
> 🔑 A line = trigger words + open hooks, not a summary.
> 🚦 Budget 16 000 bytes. `/save-progress` prints the size every run.
> 📐 The router builds its menu from **Projects** below — this is the only copy.

---

## Conventions & Behavior

- [user_profile.md](user_profile.md) — Sam: backend dev, Go/Postgres, Linux, solo
- [feedback_memory_no_bloat.md](feedback_memory_no_bloat.md) — 🌟 governance:
  core/state split · no append-growth · say it once · trigger-word index
- [feedback_no_unprompted_build.md](feedback_no_unprompted_build.md) — 🚨 in
  discussion, answer only; do not write files unless asked to build

---

## Recent Log (current month only)

> One line = date + 1–2 conclusions + link. Detail lives in the day file.

- [logs/2026-01-15.md](logs/2026-01-15.md) — 🛠️ T1: split core/state after Step 0
  flagged 18 KB · 📦 archived the Nov OCR decision
- [logs/2026-01-12.md](logs/2026-01-12.md) — 🏠 A1: NAS RAID rebuild started,
  ~40 h · blocks the Services zone until it finishes
- [logs/2026-01-08.md](logs/2026-01-08.md) — 🛠️ T1: multi-page receipts shipped ·
  📚 K1: finished ch.4 (ownership), borrow checker still fighting me

## Archived Logs

- [logs/archive.md](logs/archive.md) — 2025-10 through 2025-12

---

## Projects

> The router's menu comes from here. Numbering: category letter + serial, never
> reused. Only `core.md` is linked. `state.md` is on demand; `archived/*` when named.

### 🛠️ Tools (T)

- [T1 Receipt Scanner](projects/t1_receipt_scanner/core.md) — 🛠️ OCR receipts →
  CSV · Go + Tesseract · CLI only, never a web UI

### 🏠 Goals (A)

- [A1 Home Server Migration](projects/a1_home_server/core.md) — 🏠 off the old
  Synology · ⏳ warranty ends 2026-06 · 🚧 blocked on RAID rebuild · 🔀 zoned

### 📚 Learning (K)

- [K1 Learning Rust](projects/k1_rust/core.md) — 📚 *The Book* chapter by chapter ·
  exercise = port one T1 subcommand, not a rewrite

### ❄️ Frozen (never auto-load, never in the menu)

- ❄️ **Z1 Blog Engine** closed 2025-10-02 — replaced by a static generator; files
  at `projects/z1_blog_engine/`
