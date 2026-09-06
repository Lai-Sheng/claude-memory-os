# Memory Index

> ⚠️ **This file is loaded into every conversation.** It is an *index*, never a
> container. One line per memory file. If a line needs a second line, the content
> belongs in the file it points to.
>
> 🚦 **Hard ceiling: 16 000 bytes (~5 K tokens).** `/save-progress` measures this
> every run and triggers a slim-down when it is exceeded. See
> `feedback_memory_no_bloat.md`.

---

## Conventions & Behavior

<!-- type: user / feedback — how to work with this person. Always loaded. -->

- [user_profile.md](user_profile.md) — who the user is: role, stack, working context
- [feedback_memory_no_bloat.md](feedback_memory_no_bloat.md) — 🌟 **global memory
  governance**: every project splits into core/state; main files must not grow by
  appending; auto-split on trigger

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

> Numbering: category letter + serial. Retired numbers are never reused.
> Only `core.md` is linked here — `state.md` is read on demand and
> `archived/*` only when explicitly named.

### 🛠️ Tools (T)

- [T1 {{Project Name}}](projects/t1_{{slug}}/core.md) — 🛠️ **{{date}}**: one-line
  purpose · key constraint · where the real files live

### 🎓 Applications / Goals (A)

- [A1 {{Project Name}}](projects/a1_{{slug}}/core.md) — 🎓 **{{date}}**: purpose ·
  🚨 zone router: pick a zone before loading anything

### 📚 Learning (K)

- [K1 {{Project Name}}](projects/k1_{{slug}}/core.md) — 📚 **{{date}}**: purpose

### ❄️ Frozen (never auto-load)

> Files are kept intact. Thaw only when the user names one.

- ❄️ **{{Project}}** closed {{date}} — files at `projects/{{slug}}/`

### 📦 Handed off

- 📦 **{{Project}}** → handed to {{agent/person}} on {{date}}; see
  `{{handoff-doc-path}}`. If the user mentions it, point there instead of
  working on it here.
