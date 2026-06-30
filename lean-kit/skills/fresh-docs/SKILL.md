---
name: fresh-docs
description: Use before writing or modifying code that calls any external library, framework, SDK, CLI, or cloud API — especially versioned ones (a framework release, an SDK method, a config schema, a CLI flag). Confirms the current official API for the version in this repo instead of relying on training memory, which drifts. Use whenever you are unsure an API still exists or has the signature you remember. Do NOT use for stdlib basics or plain language syntax.
---

# Fresh docs first

Training memory goes stale; APIs rename, deprecate, and change signatures. Confirm before you commit.

## Procedure

1. **Pin the version.** Read the repo's lockfile / manifest (`package.json`, `requirements.txt`,
   `pyproject.toml`, `*.csproj`, `go.mod`, etc.) to get the *exact* version in use.
2. **Fetch the matching docs.** Open the official docs/changelog for that version — not a blog,
   not memory. If web access is available, retrieve the current page. If not, read the installed
   package's own `README` / typings / source under the dependency dir.
3. **Verify the surface you'll touch.** Signature, required args, return shape, deprecations,
   and the newest recommended idiom for that version.
4. **Then code** against what you confirmed. If docs and memory disagree, docs win.

## Stop conditions

- Can't confirm an API exists at the pinned version → don't guess. Say so and propose the
  verified alternative.
- The idiom you remember is deprecated in this version → use the current replacement.

## Note in your summary

State which version you targeted and what you verified (e.g. "confirmed against vX.Y docs:
method `foo()` now takes an options object"). One line. This is how the next agent trusts the change.
