---
name: lean
description: Apply when writing, refactoring, or reviewing any code. Enforces leverage-over-authorship — use platform, stdlib, and framework built-ins before hand-rolling; collapse duplication; keep terseness readable. Use whenever the task is to implement a feature, fix a bug, clean up, or shorten code. Do NOT use for pure prose or config-only edits.
---

# Lean code discipline

Goal: maximum capability, minimum code, zero reinvention — while staying readable.

## Before you write a single line

1. Does the language / framework / stdlib already do this? If yes, call it. Don't author it.
2. Does this shape already exist in the repo? If yes, import it. Don't duplicate it.
3. Is there a newer native idiom the repo's versions support? If yes and it's clearer, use it.

## While writing

- Declarative > imperative. Expression > statement. Composition > inheritance. Data > branching.
- One built-in beats ten hand-rolled lines — but a readable two lines beats an unreadable one.
- No speculative abstraction. Factor on the *second* duplication, not the first guess.

## Readability floor (non-negotiable)

A line that's dense but obvious is the target. A line that's dense and cryptic is a defect.
Test: could a competent teammate read it once and step through it in a debugger? If not, expand it.

## After you write

- Delete what you replaced. Dead code, unused imports, now-orphaned deps and config go too.
- Re-read the diff: every remaining line should earn its place. Cut anything that doesn't.

## When reviewing someone else's (or your earlier) code, flag

- Hand-rolled logic the platform already provides → replace with the built-in.
- Repeated near-identical blocks → parameterize into one.
- A heavy dependency doing a stdlib-sized job → drop it.
- A clever line nobody can read → expand it.

Report the *delta in lines and dependencies removed*, not just "done."
