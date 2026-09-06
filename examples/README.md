# A filled-in example

`template/` ships with 90 placeholders. That is honest, but it makes it hard to
picture what a working setup actually looks like.

This directory is the same architecture, **fully filled in** for a fictional
user: a backend developer with four projects, three months in.

Read it to see the shape. Copy from `template/`, not from here.

## What is here

```
examples/
├── CLAUDE.md                      the router, filled in — four projects
└── memory/
    ├── MEMORY.md                  the index at 2.4 KB, well under budget
    ├── user_profile.md
    ├── feedback_no_unprompted_build.md   a rule with its incident attached
    ├── feedback_memory_no_bloat.md       the governance rule, as shipped
    ├── logs/
    │   ├── 2026-01-15.md          the split happening, in a day file
    │   ├── 2026-01-12.md          a zoned project's blocker being filed
    │   ├── 2026-01-08.md          two projects in one day, separate sections
    │   └── archive.md             Oct–Dec rolled out of the index
    └── projects/
        ├── t1_receipt_scanner/    ← the fully worked one
        │   ├── core.md            spec + 4 inviolable rules, zero dates
        │   ├── state.md           current phase + resume point, overwritten
        │   └── archived/2025-11_ocr_engine_selection.md
        ├── a1_home_server/
        │   └── core.md            ← a ZONED project: the zone router
        └── k1_rust/
            └── core.md            a small one, for contrast
```

## Things worth noticing

**The router lists projects, not capabilities.** The user picks a project first.
That is what keeps the other three out of the context window.

**`core.md` has no dates in it.** Not one "completed on". Every dated line lives
in `state.md`, `archived/`, or `logs/`. That is the whole reason `core.md` stays
small enough to always load.

**`state.md` has a resume point as a single sentence.** It is the most-read line
in the file. When you come back after two weeks, it is the only line you need.

**`archived/2025-11_ocr_engine_selection.md` is a decision, not a diary.** It
records *why Tesseract lost*, so nobody re-litigates it in March. Notice it is
one themed file for a whole month — not one file per day. Days are `logs/`.

**The feedback file cites an incident.** "The user asked me to explain the retry
logic and I rewrote it instead." A rule with a story attached survives; a rule
without one gets ignored.

**The index links `core.md` and never `state.md`.** Check `MEMORY.md` — that is
the single most common thing people get wrong when adopting this.

**One log line per day.** `MEMORY.md` carries a summary; `logs/2026-01-15.md`
carries the account. Neither repeats the other.

## What this example does not show

- **The inside of a zoned project.** `a1_home_server/core.md` shows the zone
  router and the three zoning rules, but the per-zone `state.md` files and
  `facts/` are not materialised. See
  [../docs/srp-zoning.md](../docs/srp-zoning.md) for the full layout.
- **Months of accumulated feedback files.** A real setup after six months has six
  to ten. Two is enough to show the shape.
- **Every day file.** `archive.md` summarises October–December; those day files
  are not included. In a real setup they exist.
