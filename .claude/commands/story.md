---
description: >-
  Drive a story/requirement spec → implementation → release through three named,
  cost-tiered subagents (opus → sonnet → haiku), with complexity triage that
  routes cheap work down to cheaper models. Run the main session on Opus.
argument-hint: <story or requirement text>
model: opus
---

# /story — multi-model delivery pipeline

The requirement is:

> $ARGUMENTS

You are the **orchestrator**. You run on Opus (the main session). Subagent
models can only route DOWN in cost tier from the main session, so this pipeline
only works when the main session is Opus — that is the assumption here.

Your job is to deliver the requirement while spending the least money that still
gets it done correctly. You do that by pushing as much work as possible DOWN to
cheaper subagents (sonnet, then haiku), and only spending Opus where genuine
design judgment is needed.

## Dispatch rules (read carefully)

- Launch every subagent **by name**, never by auto-match. Use the Task tool with
  an explicit `subagent_type` of `spec-analyst`, `implementer`, or `releaser`.
- Run stages in **strict order** and **pass each stage's full output verbatim as
  the input to the next** stage's prompt. The implementer must receive the
  spec-analyst's spec; the releaser must receive the spec plus the implementer's
  "Notes for releaser".
- **Stop the pipeline the moment a stage does not succeed.** Surface that
  stage's output to me and do not run the later stages. Specifically:
  - spec-analyst cannot produce a coherent spec → stop.
  - implementer returns `blocked` or `failed` → stop, show its report.
  - releaser returns anything but `released` → stop, show its report.
- Do not silently skip the release. If you reach a state where there is nothing
  to release, say so explicitly.

## Step 0 — Triage (you do this yourself, on Opus, before dispatching)

First decide how big this is. Read just enough of the repo to judge, then pick a
tier. **Match the machinery to the task — do not run a 3-model pipeline for a
typo fix.**

- **trivial** — a typo, a constant, a one-line copy/config change, a rename with
  no logic. No design decisions, no real risk.
  → **Skip spec-analyst.** Hand a one-line internal spec straight to
    `implementer` (sonnet). Then `releaser` (haiku). You just saved an Opus call.

- **small** — a localized change in one or two files with clear, known behavior
  and existing test patterns. Some judgment, but the design is obvious.
  → **Skip spec-analyst.** Write a short 3–6 bullet spec yourself inline (cheap
    on Opus, but no second Opus subagent), pass it to `implementer` (sonnet),
    then `releaser` (haiku).

- **standard / large** — new behavior, multiple files, non-obvious edge cases,
  cross-cutting concerns, or anything where a wrong design wastes the cheaper
  models' effort.
  → **Run the full pipeline:** `spec-analyst` (opus) → `implementer` (sonnet)
    → `releaser` (haiku).

When unsure between two tiers, pick the cheaper one first; you can always
escalate to spec-analyst if the implementer comes back `blocked` on ambiguity.
If the spec-analyst itself classifies the work and disagrees with your triage,
trust the spec-analyst's *Complexity* field going forward.

The routing principle in one line: **the most expensive model decides what work
is worth the cheaper models' time, and nothing runs on a tier more expensive
than it needs.**

## Step 1 — Spec (opus, conditional)

For `standard`/`large`: dispatch `spec-analyst` with the requirement text.
For `trivial`/`small`: skip it; use your inline spec from triage.
If a spec cannot be formed, stop and report.

## Step 2 — Implement (sonnet)

Dispatch `implementer`. Its prompt = the spec (from spec-analyst or your inline
spec) in full. When it returns, check `Result`. If not `success`, STOP and show
me its report. If it is `blocked` specifically on a design ambiguity and you
skipped the spec-analyst, you may escalate once: run `spec-analyst` now, then
re-run `implementer` with the real spec.

## Step 3 — Release (haiku)

Dispatch `releaser`. Its prompt = the spec + the implementer's "Notes for
releaser" + confirmation that the build/tests passed. When it returns, check
`Result`. If not `released`, STOP and show me its report.

## Final report to me

End with a compact summary:
- **Tier chosen** and why (and which stages you therefore ran/skipped).
- **Spec**: one-line gist (or "inline, trivial").
- **Implementation**: files changed + test result.
- **Release**: commit message, branch, and PR/MR URL.
- **Cost note**: which stages ran on which model tier, so I can see where the
  spend went.
