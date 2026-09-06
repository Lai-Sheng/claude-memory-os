---
name: k1_rust_core
description: Read The Book chapter by chapter, porting one T1 subcommand as the exercise. An exercise, not a rewrite.
metadata:
  type: project
  tier: core
---

# K1 Learning Rust — core (permanent layer)

**Started**: 2025-12-01

---

## Why this project exists

I can read Rust and cannot write it, which means I cannot evaluate it for work.
The goal is a working mental model of ownership and lifetimes — not fluency, and
not a language migration.

## Definition of done

I can write a non-trivial CLI in Rust **without fighting the borrow checker for
more than a few minutes at a time**, and can explain to a colleague why a given
design does or does not need `Rc<RefCell<T>>`.

---

## 🔒 Inviolable rules

- 🚨 **This is an exercise, never a rewrite of T1.** The Go version stays the one
  that runs. *Why*: the fastest way to kill a working tool is to half-port it.
- 🚨 **No skipping to the interesting chapters.** *Why*: I did that with the
  async chapter, understood none of it, and had to come back through ownership
  anyway.
- 🚨 **Read → write → break.** No chapter counts as read until I have written
  something small in it and deliberately made the compiler reject it.

## Method

One chapter of *The Book* per sitting, then port one small piece of T1's
`internal/parse` into `rust-scratch/`. Ports are throwaway; the point is the
compiler errors, not the code.

> 🚨 Having generated notes or an audio summary of a chapter is **not** having
> read it. Chapter status is ⬜ until I have written code in it.
