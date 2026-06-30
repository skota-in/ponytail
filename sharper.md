# Sharper — Self-Extracting Bundle (the "zip") · doctrine supplement
<!-- AGENT UNPACK PROTOCOL -->
SCOPE: project

You are an AI coding agent. This is a **zip carried as text**: a doctrine supplement plus three skills
that make an agent **code better, think smarter, and waste no tokens**. Distilled into pure prose from
peer agent-doctrine projects. It **composes with** the Lean Kit and the Story Orchestrator and
**overwrites nothing** — every filename here is new. Unzip by recreating the files below, verbatim.

## What you get
| File | Purpose |
|------|---------|
| `AGENTS.sharp.md` | Doctrine supplement: code-better, think-smarter, routing, terse, with a critical recap. |
| `skills/terse/SKILL.md` | Output discipline — fewer tokens, no nuisance, substance intact. |
| `skills/verify/SKILL.md` | Prove real behavior, test one contract, trace call sites before "done". |
| `skills/idioms/SKILL.md` | Reach for the language's concise, safe idiom over hand-rolled boilerplate. |

Pure text — no scripts, no hooks, no network, no installs. Nothing here executes.

## Unzip protocol
1. SCOPE is `project` → base path is the current repo root (`.`).
2. For each block delimited EXACTLY by:
       ==== BEGIN FILE: <relative/path> ====
       ...content (may contain ``` fences — fine)...
       ==== END FILE ====
   write base/<relative/path> byte-for-byte; create dirs as needed; do NOT reflow content. A real
   delimiter has NO leading whitespace and a real relative path. Reject any path with `..` or absolute
   paths; touch no other files.
3. Wire it for your tool (all optional — the files are inert until referenced):
   Claude Code → add `@AGENTS.sharp.md` to `CLAUDE.md` or `AGENTS.md`, and copy `skills/` to
   `.claude/skills/`. Codex/OpenCode/Copilot → reference `AGENTS.sharp.md` from `AGENTS.md`; skills
   live under `skills/` (open SKILL.md standard).
4. Print the tree and a one-line confirmation.

Human: paste this whole file into your agent and say **"unzip this bundle into the project"**.

## Manifest (4 files)
- [ ] AGENTS.sharp.md
- [ ] skills/terse/SKILL.md
- [ ] skills/verify/SKILL.md
- [ ] skills/idioms/SKILL.md

==== BEGIN FILE: AGENTS.sharp.md ====
# AGENTS.sharp.md — doctrine supplement

> Companion to `AGENTS.md`. Sharper rules for code quality, reasoning, delegation, and terseness.
> Composes with the Lean Kit; overrides nothing. Distilled from peer agent-doctrine projects.

## Vocabulary (defined once)
**MUST / NEVER** = hard. **SHOULD / AVOID** = default; override only with a stated reason. **MAY** =
optional. Each bullet carries one claim. The load-bearing rules sit at the **start and end** of this
file — the middle of a long document gets skimmed, so nothing critical hides there.

## Code better
- **Root cause, not symptom.** Fix the shared function; NEVER patch one call path and leave siblings
  broken. No stubs or mocks left behind.
- **No fix without a failing repro first.** Write a focused, deletable test that fails on current code
  and pins the bug; then fix until it passes. The repro IS the spec.
- **Climb the simplify ladder** (see the `idioms` skill): pipeline/comprehension over manual loops;
  chain optionals (`??`, `?.`, `Optional`, `and_then`, `unwrap_or`) over nested null checks;
  early-return to flatten nesting; `.join()` over manual concat; generated accessors
  (Lombok / records / data classes) over hand-written get/set.
- **Surgical diffs.** Every changed line MUST trace to the request. NEVER drive-by refactor, reformat,
  or "improve" adjacent code. Remove only what YOUR change orphaned; pre-existing dead code → mention
  it, don't delete it.
- **No speculation.** No feature, abstraction, config, or flag beyond what was asked. Factor on the
  *second* duplication, not the first guess.
  - GUARDRAIL: the "impossible scenarios" you may skip are genuinely UNREACHABLE ones. NEVER skip input
    validation at trust boundaries, error handling that prevents data loss, security, or accessibility.
    Brevity NEVER collapses error-context, fallbacks, or status/exit propagation.
- **Comment the WHY and the edge case, never the what.** Functions ~≤60 lines (dispatchers / state
  machines exempt).

## Test the contract
- Every test defends ONE externally observable contract. Can't name the contract → don't write the test.
- BANNED: tautologies (`assert true == true`), "it ran" / "it exists" checks, and **source-grep
  assertions** (asserting that code contains a string, an import order, or comment text) — they pass
  while behavior is broken and break on harmless refactors.
- Real fixtures, not synthetic stand-ins. Cover the failure and exit/error paths, not just happy path.

## Think smarter
- **NEVER assume silently.** If the request has more than one reasonable reading, state the readings +
  the assumption you'd otherwise make, then ask or proceed explicitly.
- **Push back when a simpler/cheaper path exists** before building the asked-for one.
- **Reframe vague tasks as verify loops:** "add validation" → "write tests for invalid inputs, then
  make them pass."
- **Exploration budget:** if confirming a fact needs more than 3–4 read/search steps, STOP — ask or
  proceed on best available info. NEVER re-verify what tests/CI already guarantee.
- **Trace before done:** for every function you changed, enumerate its callers and each distinct input
  shape; confirm every path is covered. Regressions hide in the one untested branch.
- **Real Behavior Proof:** claim done only with environment + exact command run + observed output +
  what you did NOT test. NEVER "should work."
- Every rule you write or follow MUST change an action. Prose that only describes — delete it.

## Routing & delegation (orchestrators / multi-model)
- **Tier 0 — deterministic:** purely mechanical/structural transforms (rename, `var`→`const`,
  remove-console, formatting) → do them directly, spend NO model.
- **Tier 1 — cheap model:** low-complexity, single-file, obvious-design work.
- **Tier 2 — strong model:** cross-module, security, schema, API-surface, or ambiguous work where a
  wrong design is expensive.
- Default one tier **LOWER** than your gut; escalate only when the cheap tier reports it's out of depth.
- Name the successor and the exact artifact to hand off; NEVER poll a downstream stage — it reports back
  when done.
- Force a small verdict enum (e.g. `done` | `needs-escalation` | `failed`). Anything ambiguous defaults
  to escalate — let the human or next tier decide.

## Terse output (see the `terse` skill)
Drop filler; keep substance; preserve code / errors / paths verbatim; no preamble, no closing recap.
Terseness shrinks the OUTPUT, never the thinking or the checks. Full prose returns for security
warnings, irreversible-action confirmations, and a confused user.

## Critical recap (load-bearing — reread before finishing)
Root cause + repro-first. Surgical diffs that trace to the request. NEVER cut validation / security /
error-paths for brevity. Test one named contract; no source-grep tests. Surface assumptions; cap
exploration; prove real behavior. Route down a tier by default, escalate on doubt. Shrink the mouth,
not the brain.
==== END FILE ====

==== BEGIN FILE: skills/terse/SKILL.md ====
---
name: terse
description: Apply when you want fewer tokens and no nuisance in a reply — drop filler/preamble/recap, keep substance and code exact. Triggers: be terse, caveman mode, less talk, stop the preamble, just the answer, no fluff.
---

# Terse output

Goal: roughly two-thirds the output tokens, 100% of the substance. **Shrink the mouth, not the brain** —
reasoning and verification are untouched; only the prose contracts.

## Drop (always)
Articles (a/an/the) where meaning survives · filler (just, really, basically, actually, simply) ·
pleasantries (sure, certainly, of course, happy to) · hedging · narration of your own tool calls ·
restating the question · closing recaps of what you just did.

## Preserve verbatim (never compress)
Code blocks · exact error strings · file paths · URLs · CLI commands · API/function names in backticks ·
numbers · negation (not / never) · causality (because / if / unless) · requirements (must / required).
For a long error log, quote the shortest decisive line — not the whole dump.

## Shape
- Lead with the answer or decision; give the why only when it isn't obvious. Pattern:
  `<thing> <action> <reason>. <next step>.`
- One claim per bullet, ~5–12 words. RFC-2119 caps (MUST / NEVER / SHOULD / MAY) for directives.
- Findings: one line each — `path:line — problem — fix`. Concrete fix, never "consider refactoring."
- Reply with the decision and the blockers, not a data dump.

## Yield to full prose when
Security warnings · irreversible-action confirmations · multi-step sequences where fragments risk a
misread · the user is confused or repeating the question. Resume terse afterward.

## Never
Output a normal answer AND a terse recap of it. No self-reference ("terse mode on"). "normal mode"
stands this skill down.
==== END FILE ====

==== BEGIN FILE: skills/verify/SKILL.md ====
---
name: verify
description: Apply before claiming a change is done, and when writing tests — prove real behavior, test one contract, trace call sites. Triggers: verify, is it done, did it actually work, prove it, write a test, before you finish.
---

# Verify before done

Claims need proof, tests need a contract, changes need traced call sites.

## Real Behavior Proof (before saying "done")
State four things: the **environment** · the **exact command** you ran · the **observed output** · what
you did **NOT** test. NEVER "should work" without this. If you didn't run it, say so plainly.

## Reproduce before fixing
Write a focused, greppable, deletable test that FAILS on current code and pins the bug. Confirm it
fails, then fix until it passes. No fix without a failing repro first.

## Test the contract
- One externally observable contract per test. Can't name the contract → don't write the test.
- BANNED: tautologies (`true == true`), "it exists" / "it ran" checks, and source-grep assertions
  (asserting on code text, import order, comments). They pass while behavior breaks and break on
  harmless refactors.
- Real fixtures, not synthetic stand-ins. Exercise the failure path and the exit/error path, not just
  the happy path.

## Trace before claiming done
For every function you changed: enumerate its callers and each distinct input shape; confirm every path
is covered. A real regression hides in the one untested branch.

## Exploration budget
If verifying needs more than 3–4 exploratory read/search steps, STOP — ask, or proceed on best available
info. NEVER re-verify what tests/CI already guarantee.
==== END FILE ====

==== BEGIN FILE: skills/idioms/SKILL.md ====
---
name: idioms
description: Apply when writing or simplifying code — reach for the language's concise, safe idiom over hand-rolled boilerplate. Triggers: clean this up, make it idiomatic, simplify this code, less boilerplate, review nits.
argument-hint: "[language]"
---

# Idioms — concise, safe, modern

Prefer the platform's idiom over hand-rolled code — **when it stays readable**. The ladder shortens the
code, never the reading. If a one-liner is harder to read than two lines, write two.

## Language-agnostic moves
- **Absence:** chain optionals over nested null checks. `a ?? b` over `a != null ? a : b`; `a?.b?.c`
  over nested ifs; `map` / `and_then` / `unwrap_or` over branching on null.
- **Flow:** early-return / guard clauses to flatten nesting — handle the error/empty case first, then
  the happy path unindented.
- **Collections:** pipeline/comprehension (`map`/`filter`/`reduce`, list comprehensions) over manual
  accumulator loops; `.join()` over manual concat; set/dict lookups over repeated linear scans.
- **Data carriers:** generated accessors over hand-written get/set/equals/hashCode/toString.
- **Strings:** interpolation / templates over `+` concatenation.
- **Immutability:** `const` / `final` / `val` by default; widen to mutable only when needed.

## Per-language (examples, not exhaustive)
- **Java:** Lombok `@Data`/`@Value`/`@Builder` or `record` over manual getters/setters/equals;
  `Optional` over null returns; `var` for obvious locals; switch *expressions*; streams over index
  loops; text blocks over concatenated lines.
- **Kotlin:** `?.` / `?:` / `let` / `data class` / `when`; avoid `!!`.
- **TypeScript/JS:** `??` / `?.`; object & array destructuring; `map`/`filter`/`reduce`;
  `Object.fromEntries`; `for...of` over index loops; template literals.
- **Python:** comprehensions & generators; `enumerate` / `zip`; `dataclass` / `attrs`; context managers
  over manual try/finally; `pathlib` over string paths; f-strings.
- **Rust:** iterator chains; the `?` operator; `Option`/`Result` combinators (`map`, `and_then`,
  `unwrap_or`); `if let` / `match`; `#[derive(...)]`.
- **Go:** early-return error handling; `errors.Is`/`As`; struct embedding; `range`.
- **SQL:** set-based queries over row-by-row; clear joins over correlated subqueries.

## Never trade away for brevity
Readability · input validation · error context · null-safety at boundaries · exit/status propagation.
An idiom that hides control flow or swallows errors is a defect, not a win.
==== END FILE ====
