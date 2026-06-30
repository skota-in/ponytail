---
name: review
description: Review a diff or a whole repo for over-engineering and reinvention before it lands — reinvented stdlib/platform features, needless dependencies, speculative abstractions, duplicated shapes, versioned APIs coded from stale memory, and clever-but-unreadable lines. Reports what to delete or simplify, one line per finding; does not edit. Use on a PR, a staged diff, a freshly written file, or the whole tree. Triggers on "review for over-engineering", "what can we delete", "is this over-engineered", "simplify review", "audit this codebase", "find bloat".
---

# Lean review

Review for unnecessary complexity and reinvention. One line per finding: location, what to cut,
what replaces it. The best outcome is the change getting **shorter**. Reports only — applies nothing.

## Scope

- A **diff** (default): the staged/PR change.
- A **whole repo** ("audit"): scan the tree, rank findings biggest-cut-first.

Out of scope: correctness bugs, security holes, performance — route those to a normal review pass.
A single smoke test or `assert`-based self-check is the lean minimum, **not** bloat — never flag it
for deletion.

## Format

`L<line>: <tag> <what>. <replacement>.` — or `<file>:L<line>: …` for multi-file diffs / audits.

## Tags

- `delete:` dead code, unused flexibility, speculative feature. Replacement: nothing.
- `stdlib:` hand-rolled thing the language/standard library ships. Name the function.
- `native:` dependency or code doing what the platform already does. Name the feature.
- `yagni:` abstraction with one implementation, config nobody sets, a layer with one caller.
- `dupe:` shape repeated from elsewhere in the repo or within the diff. Point to the one to import
  or the block to parameterize.
- `stale:` a versioned API call that looks written from memory. Flag for a `fresh-docs` check.
- `shrink:` same logic, fewer lines / a clearer idiom. Show the shorter form.

## Examples

- `L12-38: stdlib: 27-line email validator class. "@" + a dot check is 1 line; real validation is the confirmation mail.`
- `L4: native: moment.js imported for one format call. Intl.DateTimeFormat, 0 deps.`
- `repo.py:L88: yagni: AbstractRepository with one implementation. Inline it until a second exists.`
- `api.ts:L21: stale: client.send() — signature changed in v3; confirm against current docs.`
- `L30-44: shrink: manual loop builds a dict. Object.fromEntries(pairs), 1 line.`

## Output

Lead with the highest-leverage finding (usually the biggest deletion). End with the only metric
that matters: `net: -<N> lines, -<M> deps possible.` Nothing to cut: `Lean already. Ship.`
No praise padding, no restating the diff.

"normal mode" reverts to a verbose review style.
