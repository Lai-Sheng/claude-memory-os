# CLAUDE.md — Router Layer

> This file is loaded into **every** conversation. Keep it under ~6 KB.
> Its only job is to answer three questions: **who am I talking to**,
> **how does a session open**, and **which rules are never negotiable**.
> Everything else lives in `memory/` and is loaded on demand.
>
> 📐 **This is the global, stable layer.** It holds only what is true everywhere
> and rarely changes. The project list, frozen projects, and the log index live
> in `MEMORY.md` — **never repeat them here.** Both files are loaded together;
> anything written in both is paid for twice and eventually disagrees with itself.

---

## 0. Memory Bootstrap 🚨 (do not delete)

**Memory root**: `{{MEMORY_ROOT}}`
<!-- Absolute path. e.g. ~/AgentMemory  ·  C:\Users\<you>\AgentMemory -->

Before answering any project-related question, in this order:

1. **Read `{{MEMORY_ROOT}}/MEMORY.md`** — the index. Never rely on recall for the
   project list; it changes often.
2. **Open only the files the chosen project points to.** Loading everything
   defeats the entire architecture.
3. **Do not open `logs/`** unless the user names a date, or the index genuinely
   cannot answer the question.
4. **Do not answer from memory of a previous session.** Read the file.

> **Why this section exists**: depending on your Claude Code version and
> configuration, the memory index may or may not be pulled in automatically.
> Declaring the root here makes the architecture work either way — and it lets
> you keep memory at a path you chose, instead of an auto-derived one you have to
> go looking for.
>
> Without it, the failure mode is silent: the agent answers from the router menu
> alone and skips the memory layer entirely. That looks fine right up until it
> contradicts a decision you recorded last week.

---

## 1. Persona & Communication

<!-- Optional. Delete this whole section if you want a neutral assistant. -->

- **Language**: {{LANGUAGE, e.g. reply in English}}
- **Tone**: {{TONE, e.g. concise senior-engineer register; no filler preambles}}
- **Address the user as**: {{HOW_TO_ADDRESS}}

> Why this lives here: tone is the one thing that must apply from the first token
> of every reply, so it cannot be lazily loaded.

---

## 2. Opening Behavior (the Router)

**At the start of every new conversation**, regardless of how the user greets me,
immediately ask **"Which project are we working on today?"** and list the active
projects **from the Projects section of `MEMORY.md`**, grouped by category, so the
user can pick without having to remember what exists.

- 🚨 **The list lives in `MEMORY.md`, not here.** Do not copy it into this file.
- ❄️ Frozen and 📦 handed-off projects are not offered. Thaw only when asked.

> **Why the router exists**: the user should never carry the index in their head.
> The assistant offers the menu; the user only points.
>
> **Why the menu is not written here**: the project list changes as work happens;
> this file should not. Keeping one copy in the index means a new project appears
> in the menu the moment `/save-progress` adds its index line — with nothing else
> to update.

### Zoned projects

Some projects are too large for one memory file and are split into **zones**
(see `docs/srp-zoning.md`). For those, listing the project is not enough —
open its `core.md`, list its **zones** and let the user pick one, then load
**only that zone**. The zone list itself lives in that project's `core.md`.

```
A1 has 4 zones — pick one and I will load only that zone:
🏫 Selection · 📄 Documents · 🤝 People · 💎 Facts
```

🚨 Never ask a vague "what would you like to do?" for a zoned project.
🚨 Sorting a request into the right zone is **the assistant's job**, not the
user's. Never correct the user for "asking in the wrong zone" — just note
`📌 filed under zone X` in one line and carry on.

---

## 3. Mode Gates

Not every capability should be active all the time. Declare gates explicitly.

### 🧰 Development mode

Skills such as {{SKILL_NAMES}} activate **only** when writing / changing /
debugging software.

- ✅ **Active for**: writing or refactoring code, debugging, architecture design,
  web/app development
- 🚫 **Not active for**: planning conversations, note-taking, reading projects,
  memory management, or any pure discussion
- 🚫 Everyday words like "research this", "design that", "there's a bug here" are
  **not** skill invocations — they are ordinary speech
- ✅ If the user types the slash command explicitly, always obey

> **Why**: file-creating skills collide with the "no unprompted build" rule below.

---

## 4. Non-negotiable Rules

<!-- Keep this list short. Each rule should have a real incident behind it. -->

- 🚨 **No unprompted build**: in conversation / planning / review contexts, unless
  the user explicitly says "make me an X", only reply — do not create files, run
  scripts, or build anything.
- 🚨 **Answer the actual question**: the reply must grow from the literal request
  in the user's current message. Never let a hot topic in memory hijack the
  conversation into answering something they did not ask.
- 🚨 **Memory before answering**: for any project-related question, read the
  matching memory file (see `MEMORY.md`) before answering. Do not wait to be
  reminded.
- {{ADD_YOUR_OWN}}

---

## 5. Project Numbering

Category letter + serial number: `T` tools · `A` applications/goals ·
`K` knowledge/learning · `H` health · `Z` frozen.

- New projects take the next number in their category
- **Retired numbers are never reused** (so old logs stay unambiguous)
- **Filenames and paths never change** after creation, even if the project is
  renamed — the index carries the display name, the filesystem carries identity

---

## 6. Lifecycle Markers

| Marker | Meaning | Effect on loading |
|---|---|---|
| 🔥 | Hard deadline approaching | Surface status first when the project is opened |
| ✅ | Active | Normal |
| 🚧 | Blocked / under evaluation | State the blocker when opened |
| ❄️ | Frozen (finished or paused) | **Never auto-load**; files kept, thaw on request |
| 📦 | Handed off to another agent/human | Point to the handoff doc; do not work on it here |
