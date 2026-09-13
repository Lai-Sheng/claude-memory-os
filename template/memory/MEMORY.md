# Memory Index

> ⚠️ **This file is loaded into every conversation.** It is an *index*, never a
> container. One line per memory file. If a line needs a second line, the content
> belongs in the file it points to.
>
> 🔑 **A line is a trigger, not a summary**: keywords + any open hook (*blocked on
> X*, *decision pending*). No explanation. ~120–180 bytes.
>
> 🚦 **Budget: 16 000 bytes (~5 K tokens).** `/save-progress` prints this file's
> size on every run. See `feedback_memory_no_bloat.md`.
>
> 📐 This file owns what *changes*: the project list (the router's menu is built
> from it), freeze state, the log index. Global, stable rules live in `CLAUDE.md`.
> **Never write the same fact in both.**

---

## Conventions & Behavior

<!-- type: user / feedback — how to work with this person. Always loaded. -->

- [user_profile.md](user_profile.md) — who the user is: role, stack, working context
- [feedback_memory_no_bloat.md](feedback_memory_no_bloat.md) — 🌟 memory governance:
  core/state split · no append-growth · trigger-word index · say it once

<!-- Add one line per behavioral rule you have learned. Examples:
- [feedback_no_clarifying_questions.md](feedback_no_clarifying_questions.md) — decide at forks instead of asking
- [feedback_pricing_currency.md](feedback_pricing_currency.md) — always quote in local currency
-->

---

## Recent Log (current month only)

> One line per day = date + 1–2 conclusions + link. The play-by-play lives in
> `logs/YYYY-MM-DD.md`. **Writing more than one line here is how bloat starts.**
> At the start of each month, move the previous month's lines into
> `logs/archive.md` and leave a single pointer.

- [logs/2026-01-15.md](logs/2026-01-15.md) — 🏗️ example: split project X into core/state · 📖 example: 3 decisions logged

## Archived Logs

- [logs/archive.md](logs/archive.md) — everything before the current month

---

## Projects

> **The router's opening menu is built from this section — it is the only copy.**
> Numbering: category letter + serial. Retired numbers are never reused.
> Only `core.md` is linked here — `state.md` is read on demand and
> `archived/*` only when explicitly named.

### 🛠️ Tools (T)

- [T1 {{Project Name}}](projects/t1_{{slug}}/core.md) — 🛠️ {{purpose keywords}} ·
  {{key constraint}} · 🚧 {{open hook, if any}}

### 🎓 Applications / Goals (A)

- [A1 {{Project Name}}](projects/a1_{{slug}}/core.md) — 🎓 {{purpose keywords}} ·
  🔀 zoned — pick a zone before loading anything

### 📚 Learning (K)

- [K1 {{Project Name}}](projects/k1_{{slug}}/core.md) — 📚 {{purpose keywords}} ·
  {{where you are}}

### ❄️ Frozen (never auto-load, never in the menu)

> Files are kept intact. Thaw only when the user names one.

- ❄️ **{{Project}}** closed {{date}} — files at `projects/{{slug}}/`

### 📦 Handed off (not in the menu)

- 📦 **{{Project}}** → handed to {{person/tool}} on {{date}}; see
  `{{handoff-note-path}}`. If the user mentions it, point there instead of
  working on it here.
