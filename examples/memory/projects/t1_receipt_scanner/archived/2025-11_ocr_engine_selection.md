# T1 · archived — OCR engine selection (2025-11)

> Archived state. Kept because it records **why**, so the decision does not get
> re-litigated. One themed file for the month — days belong in `logs/`.

## Decision

**Tesseract 5 with a custom `--psm 6` profile**, running locally.

## What was compared

| Engine | Clean parse (100-receipt sample) | Cost | Verdict |
|---|---|---|---|
| Tesseract 5 (local) | 87 % | £0 | ✅ chosen |
| Cloud vision API | 96 % | ~£14/quarter | ❌ rejected |
| Paddle OCR (local) | 89 % | £0 | ❌ rejected |

## Why the most accurate option lost

The cloud API was clearly the best at reading thermal paper — 9 points ahead, and
it handled the Costa receipts that still fail today. It was rejected anyway:

- **Receipts are financial records for a real business.** Shipping them to a
  third party for a 9-point gain is a privacy trade I am not willing to make on
  someone else's data.
- The gap closes in `parse`, not in `ocr`. Most of the 13 % were **layout**
  failures, not character failures — Tesseract read the digits fine and the parse
  rules picked the wrong line. That is a bug I can fix; a cloud dependency is not.

Paddle scored 2 points above Tesseract but needed a Python runtime alongside the
Go binary. Not worth a second toolchain for 2 points.

## What would reopen this

- Tesseract stalls below ~95 % clean parse **after** the parse rules are
  exhausted — i.e. the remaining failures are genuinely character-level
- A local model reaches cloud accuracy with no external calls

## Still true today

The `ocr` package deliberately exposes only `Recognize(image) (text, confidences)`.
Swapping the engine should be one file. That seam was designed during this
comparison and is now recorded in `core.md`.
