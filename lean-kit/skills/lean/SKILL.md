---
name: lean
description: Apply when writing, refactoring, or reviewing any code. Enforces leverage-over-authorship and the lazy-senior-dev ladder — understand the problem, then use platform/stdlib/framework built-ins before hand-rolling, reuse before rewriting, one line before fifty, while keeping terseness readable and never cutting validation, security, or accessibility. Use whenever the task is to implement a feature, fix a bug, clean up, or shorten code. Triggers also on "be lazy", "lean mode", "simplest solution", "minimal", "yagni", "do less", "shortest path", or complaints about over-engineering, bloat, boilerplate, or needless dependencies. Do NOT use for pure prose or config-only edits.
argument-hint: "[lite|full|ultra]"
---

# Lean code discipline

Goal: maximum capability, minimum code, zero reinvention — while staying readable and correct.
The companion enforcement of the doctrine in `AGENTS.md`.

## First, understand — then climb the ladder

Read the task and the code it touches; trace the real flow end to end. *Then* stop at the first
rung that holds:

1. **Need it at all?** Speculative → skip it, say so in one line. (YAGNI)
2. **Already in this repo?** Reuse the helper / util / type / pattern. Don't re-author it.
3. **Stdlib / language does it?** Call it.
4. **Native platform feature covers it?** Use it (`<input type="date">`, CSS over JS, DB
   constraint over app code).
5. **Installed dependency solves it?** Use it. No new dep for what a few lines do.
6. **One line?** One line — if it stays readable.
7. **Only then:** the minimum that works.

Two rungs work → take the higher and move on. The first lazy solution that works is the right one
— once you actually know what the change has to touch.

**Bug fix = root cause.** Grep every caller of the function you touch; fix the shared function
once. Patching only the path the ticket names leaves sibling callers broken.

## While writing

- Declarative > imperative. Expression > statement. Composition > inheritance. Data > branching.
- One built-in beats ten hand-rolled lines — but a readable two lines beats an unreadable one.
- No speculative abstraction: no interface with one impl, no factory for one product, no config for
  a value that never changes. Factor on the *second* duplication, not the first guess.

## Readability floor (non-negotiable)

Dense but obvious is the target; dense and cryptic is a defect. Test: could a competent teammate
read it once and step through it in a debugger? If not, expand it.

## Never lazy about

Input validation at trust boundaries, error handling that prevents data loss, security,
accessibility, hardware calibration, and anything the user explicitly asked for. Non-trivial logic
leaves ONE runnable check behind (an assert-based self-check or one small test — no frameworks).
Trivial one-liners need no test.

## After you write

- Delete what you replaced — dead code, unused imports, now-orphaned deps and config go too.
- Mark deliberate shortcuts with a `lean:` comment naming the ceiling and upgrade path.
- Re-read the diff: every remaining line earns its place. Cut what doesn't.

## Output

Code first, then at most three lines: what was skipped, when to add it. If the explanation is
longer than the code, delete the explanation. Report the *delta in lines and dependencies removed*,
not just "done."

## Intensity

- **lite** — build what's asked; name the lazier alternative in one line.
- **full** *(default)* — the ladder enforced; stdlib and native first; shortest diff and explanation.
- **ultra** — YAGNI extremist; deletion before addition; ship the one-liner and challenge the rest
  of the requirement in the same breath.

"normal mode" stands the discipline down for a task.
