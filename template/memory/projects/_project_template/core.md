---
name: {{project_id}}_core
description: {{One line. This is what the assistant reads in the index to decide relevance — make it specific.}}
metadata:
  type: project
  tier: core
---

# {{Project Name}} — core (permanent layer)

**Started**: {{YYYY-MM-DD}}

> 🚦 **core.md holds only what is still true in a year.**
> No "completed on YYYY-MM-DD" sections. No instance IDs, file mtimes, version
> numbers, or other perishable snapshots. Those belong in `state.md` or `logs/`.
> This file changes only on a **strategic pivot**.

---

## Why this project exists

{{One paragraph. The problem, not the solution. When a decision is contested
later, this is the paragraph that settles it.}}

## Definition of done

{{What "finished" means. A project without this never ends.}}

---

## 🔒 Inviolable rules

<!-- Each rule should trace back to a real incident or a deliberate decision. -->

- 🚨 {{Rule}} — *why*: {{reason}}
- 🚨 {{Rule}} — *why*: {{reason}}

---

## Architecture / structure

{{Design constitution, mechanics list, spec, module map — whatever this project's
permanent shape is.}}

---

## Where the real files live

| What | Path |
|---|---|
| Source / working files | `{{path}}` |
| Deliverables the user opens themselves | `{{path}}` |
| Notes in an external tool | `{{path or app}}` |

> Memory files describe and route. They are **not** the artifact store.

---

## Zone router (delete if this project is not zoned)

> Only for projects too large for one file. See `docs/srp-zoning.md`.
> When the user opens this project, **list the zones and let them pick**, then
> load only that zone.

| Zone | Owns | File |
|---|---|---|
| 🏫 {{Zone A}} | {{single responsibility}} | `{{zone_a}}/state.md` |
| 📄 {{Zone B}} | {{single responsibility}} | `{{zone_b}}/state.md` |
| 💎 Facts | Verified facts every zone needs | `facts/*.md` |
| 📥 Inbox | Incoming external material, sorted on arrival | `inbox/*.md` |

**Zoning rules**
1. **Facts sink** — a verified fact lives in `facts/`, once
2. **State belongs to whoever drives it** — not to whoever mentioned it
3. **Cross-zone links only, never copies** — reference `[[the-other-file]]`

---

## Related memory

- `[[{{other-file}}]]` — {{relation}}
