---
name: t1_receipt_scanner_core
description: CLI that OCRs paper receipts into the CSV shape the accountant needs. Go + Tesseract. Permanent layer — spec and inviolable rules.
metadata:
  type: project
  tier: core
---

# T1 Receipt Scanner — core (permanent layer)

**Started**: 2025-10-14

> 🚦 Nothing dated lives here. No "completed on". No versions, no file sizes.
> Those go in `state.md`, `archived/`, or `logs/`.
> This file changes only on a strategic pivot.

---

## Why this project exists

Every quarter I hand my accountant a shoebox of paper receipts and she charges me
for two hours of typing. The scanner turns that pile into the exact CSV her
software imports, so the quarterly handover costs me ten minutes and her nothing.

The point is **her import format**, not OCR. OCR is the hard part; the format is
the deliverable. Anything that improves recognition but breaks the CSV shape is a
regression.

## Definition of done

A quarter's receipts go from a scanned folder to an accepted CSV import with **no
manual correction on the merchant, date, total or VAT fields**. Line items are
out of scope — she does not use them.

When that holds for two consecutive quarters, this project is finished and gets
frozen.

---

## 🔒 Inviolable rules

- 🚨 **CLI only. Never a web UI.** *Why*: I proposed one in November and talked
  myself out of it — a UI would be a second project with auth, hosting and
  upkeep, and it does not make the CSV any more correct.
- 🚨 **The CSV schema is owned by the accountant, not by me.** *Why*: I "improved"
  the column order once and her import silently dropped VAT on 40 rows. The
  schema is in `facts/csv_schema.md` and changes only when she sends a new spec.
- 🚨 **Never mutate the source images.** Read-only, always write to a copy.
  *Why*: an early rotation pass overwrote 200 originals with a bad deskew and
  they were not recoverable.
- 🚨 **Totals are parsed, never inferred.** If the total cannot be read with
  confidence, flag the row. *Why*: a plausible wrong number is worse than a blank
  — she checks blanks and trusts numbers.

---

## Architecture

```
cmd/scan/          entry point, flag parsing
internal/ingest/   walk folder, dedupe by content hash, normalise to PNG
internal/ocr/      Tesseract wrapper, one image → raw text + confidences
internal/parse/    raw text → Receipt struct (merchant, date, total, VAT)
internal/export/   []Receipt → the accountant's CSV
```

**The seam that matters** is `parse`. `ocr` hands it text and confidences and
knows nothing about receipts; `parse` knows nothing about images. Every merchant
quirk lives behind that boundary as a rule in `internal/parse/rules/`, so a new
supermarket format is one file and one test, never a change to the pipeline.

## Where the real files live

| What | Path |
|---|---|
| Source | `~/code/receipt-scanner` |
| Scanned input (read-only) | `~/Documents/receipts/<quarter>/` |
| Output CSVs | `~/Documents/receipts/export/` |
| The accountant's schema | `facts/csv_schema.md` |

---

## Related memory

- `[[feedback-no-unprompted-build]]` — this project is where that rule was earned
