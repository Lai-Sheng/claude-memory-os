---
name: t1_receipt_scanner_state
description: Current in-flight work. Overwritten each phase, never appended. Not linked from MEMORY.md.
metadata:
  type: project
  tier: state
---

# T1 Receipt Scanner — state

**Last updated**: 2026-01-15

> ✍️ Overwritten, not appended. Before adding a line: *will this matter in 30
> days?* No → it belongs in `logs/` only. Superseded content → `archived/`.

---

## 🎯 Right now

Q4 2025 is the first quarter running end to end without hand-editing. 214 of 231
receipts parsed clean; the 17 failures are all one merchant (Costa, thermal
paper, the total sits under a logo that survives deskew as noise).

## ⏭️ Resume point

`internal/parse/rules/costa.go` — the rule matches the receipt but grabs the
*subtotal* line because both are prefixed `TOTAL`. Anchor on the last match in
the block rather than the first; there is a failing test at
`parse/rules/costa_test.go:41` that reproduces it.

## 🚧 Blockers

| Blocker | Blocking what | Owner | Since |
|---|---|---|---|
| Waiting on the accountant to confirm VAT column for zero-rated items | Q4 export sign-off | her | 2026-01-09 |

## 📋 Open TODOs

- [ ] Fix the Costa subtotal/total anchor (see resume point)
- [ ] Re-run Q4 after the fix and diff against the hand-corrected CSV — must be
      byte-identical on merchant/date/total/VAT
- [ ] Add a `--flag-only` mode so low-confidence rows come out blank instead of
      guessed (rule 4 in `core.md` is currently enforced by hand)

## 🗓️ Recent milestones (last ~30 days)

- **2026-01-15** — split this project into core/state; Nov OCR decision archived
- **2026-01-08** — multi-page receipts join into one record (content-hash dedupe)
- **2025-12-20** — Q4 dry run: 92.6 % clean parse, all failures one merchant

## ❓ Open questions

- **Do I add line-item extraction?** She does not use it today, but her new
  software might. Cost: roughly doubles the parse surface. Leaning **no** until
  she asks — but it is her call, not mine.
