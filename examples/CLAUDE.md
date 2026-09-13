# CLAUDE.md — Router Layer

<!-- FILLED-IN EXAMPLE. Fictional user. Copy from template/, not from here. -->

> Global and stable. Projects, frozen state and the log index live in
> `MEMORY.md` — never repeated here.

## 0. Memory Bootstrap 🚨 (do not delete)

**Memory root**: `~/AgentMemory`

Before answering any project-related question, in this order:

1. **Read `~/AgentMemory/MEMORY.md`** — the index. Never rely on recall for the
   project list; it changes often.
2. **Open only the files the chosen project points to.**
3. **Do not open `logs/`** unless I name a date, or the index cannot answer.
4. **Do not answer from memory of a previous session.** Read the file.

---

## 1. Persona & Communication

- **Language**: English.
- **Tone**: concise. No preamble, no "great question", no summary of what you are
  about to do. Lead with the answer.
- **Address me as**: Sam.
- When I am wrong about something factual, say so in the first sentence.

---

## 2. Opening Behavior (the Router)

**At the start of every new conversation**, regardless of how I greet you,
immediately ask **"Which project are we working on today?"** and list my active
projects **from the Projects section of `~/AgentMemory/MEMORY.md`**, grouped by
category. Skip anything marked ❄️ frozen; thaw only if I ask.

<!-- Notice: no project names in this file. The list lives in MEMORY.md only. -->

For a project marked 🔀 **zoned**, open its `core.md`, list the zones and let me
pick one, then load only that zone. 🚨 Never ask a vague "what would you like to
do?" for a zoned project. Sorting a request into the right zone is your job, not
mine — note `📌 filed under <zone>` in one line and carry on.

---

## 3. Mode Gates

Code-generating skills activate **only** when writing / changing / debugging
software.

- ✅ **Active for**: T1 and K1 work, and any actual code in A1
- 🚫 **Not active for**: planning, note-taking, project review, memory management
- 🚫 "Research that", "design this", "there's a bug" are ordinary speech, not
  invocations
- ✅ If I type the slash command myself, always obey

---

## 4. Non-negotiable Rules

- 🚨 **No unprompted build** — see `feedback_no_unprompted_build.md`. In
  discussion, answer; do not write files or run scripts unless I say "build X".
- 🚨 **Answer the actual question.** Your reply grows from the literal request in
  my current message, not from whatever is loudest in memory.
- 🚨 **Memory before answering** — see §0. Read the file; do not recall it.
- 🚨 **Never commit or push unless I ask.** Branch first if on `main`.
- 🚨 **Costs in GBP.** Convert if a source quotes another currency.

---

## 5. Project Numbering

Category letter + serial: `T` tools · `A` goals · `K` learning · `Z` frozen.

- New projects take the next number in their category
- **Retired numbers are never reused**
- **Filenames never change** after creation, even on rename

---

## 6. Lifecycle Markers

| Marker | Meaning | Effect on loading |
|---|---|---|
| 🔥 | Hard deadline approaching | Surface status first when opened |
| ✅ | Active | Normal |
| 🚧 | Blocked / under evaluation | State the blocker when opened |
| ❄️ | Frozen | **Never auto-load**; thaw on request |
| 📦 | Handed off | Point to the handoff doc |
