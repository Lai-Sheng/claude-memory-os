---
name: {{project_id}}_state
description: Current in-flight work. OVERWRITTEN each phase — never appended. Not linked from MEMORY.md; read on demand.
metadata:
  type: project
  tier: state
---

# {{Project Name}} — state

**Last updated**: {{YYYY-MM-DD}}

> ✍️ **This file is overwritten, not appended.** Before adding a line, ask:
> *"will this still matter in 30 days?"*
> Yes → it belongs here. No → it belongs in `logs/YYYY-MM-DD.md` only.
> Superseded content moves to `archived/YYYY-MM_topic.md`.

---

## 🎯 Right now

{{Two or three sentences: what phase this is in and what it is waiting on.}}

## ⏭️ Resume point

{{The single most useful sentence in this file — exactly where to pick up.}}

## 🚧 Blockers

| Blocker | Blocking what | Owner | Since |
|---|---|---|---|
| {{...}} | {{...}} | {{user / assistant / external}} | {{date}} |

## 📋 Open TODOs

- [ ] {{...}}
- [ ] {{...}}

## 🗓️ Recent milestones (last ~30 days)

- **{{YYYY-MM-DD}}** — {{outcome, not activity}}

## ❓ Open questions / undecided

> Decisions the assistant must **not** make on the user's behalf. List the
> options and the reasoning; leave the call to them.

- **{{Question}}** — options: {{A}} / {{B}}; trade-off: {{...}}
