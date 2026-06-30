# Lean Kit — portable AI-dev doctrine + self-extracting bundles

A tiny, tool-agnostic kit that makes any coding agent (Codex, Claude Code, Copilot, OpenCode,
Cursor, Gemini) write **less code that does more** — and that travels as plain Markdown so you
can recreate it even where GitHub clones and zip downloads are blocked.

## Two things in one

1. **The doctrine** — `AGENTS.md` is the single source of truth: leverage built-ins over
   hand-rolled code, confirm versioned APIs against current docs, keep terseness readable, never
   repeat the codebase. Every tool reads it (or a one-line pointer to it). Skills, subagents, and
   hooks enforce it automatically.
2. **The bundle** — `BOOTSTRAP.md` is a *self-extracting* Markdown: it carries every file above as
   text plus instructions any agent follows to write them back to disk. Paste it into Codex/Claude/
   Copilot/OpenCode and say "unpack this." That's the zip/unzip, with no download.

## Why one source of truth

`SKILL.md` and `AGENTS.md` are now cross-tool open standards — the same files work in Claude Code,
Codex CLI, GitHub Copilot, OpenCode, Cursor, and Gemini. So the rules live **once** in `AGENTS.md`;
the per-tool files are thin pointers (`CLAUDE.md` is literally `@AGENTS.md`). Edit one file, every
agent updates. No copy-paste drift.

## Where files go, per tool

| Piece | Cross-tool source | Claude Code | Copilot | Codex / OpenCode |
|---|---|---|---|---|
| Doctrine | `AGENTS.md` (root) | `CLAUDE.md` → `@AGENTS.md` | `.github/copilot-instructions.md` (+ reads `AGENTS.md`) | reads `AGENTS.md` |
| Skills | `skills/<n>/SKILL.md` | `.claude/skills/<n>/` | `.github/skills/<n>/` | `~/.codex/skills/` or repo skills dir |
| Subagents | `agents/<n>.md` | `.claude/agents/` | `.github/agents/<n>.agent.md` | tool's agent dir |
| Hooks | `.claude/settings.json` | native | `.github/hooks/` | tool's hook config |

Skills/agents use the open frontmatter (`name`, `description`), so they drop into any tool's dir
unchanged. Claude-only frontmatter (e.g. `context: fork`, `tools`) is safely ignored elsewhere.

## Per-project vs per-user

- **Per-project:** drop these files at the repo root / its `.claude`, `.github`. Scope the bundle
  with `SCOPE: project`. The rules apply to that repo only.
- **Per-user (global default for all your repos):** put the same files under your home config —
  `~/.claude/` (Claude Code), `~/.codex/` (Codex), `~/.copilot/` (Copilot), `~/.config/opencode/`.
  Scope the bundle with `SCOPE: user`. Project files still override user files where both exist.

## Using the bundle (the zip/unzip)

**Unzip** — in any agent: *"Unpack this bundle into the project"* and paste `BOOTSTRAP.md`
(or the contents of any `*.bundle.md`). The agent reads the header, resolves `SCOPE`, and writes
every file verbatim. The `unpack` skill makes agents that have it do this automatically.

**Zip** — *"Pack this project into a shareable bundle"*. The `pack` skill / `pack.mjs` serializes
the chosen files into one Markdown with `==== BEGIN FILE ====` markers. Share the text; no download.

```
node pack.mjs            # regenerate BOOTSTRAP.md from the current files (project scope)
node pack.mjs --user     # emit a user-scoped bundle
```

## One honest note

"Less code" is a means, not the goal. The win is **leverage** — letting the platform do the work —
not code golf. The doctrine encodes a hard readability floor for exactly this reason: a one-liner
that's harder to read than two lines is a defect, not a flex. Densify *after* it's correct and clear.

## Files

```
AGENTS.md                         # doctrine — the source of truth
CLAUDE.md                         # @AGENTS.md (Claude pointer)
.github/copilot-instructions.md   # Copilot pointer
.claude/settings.json             # hooks: auto-format + verify gate
skills/lean/SKILL.md              # leverage / anti-boilerplate discipline
skills/fresh-docs/SKILL.md        # confirm versioned APIs against current docs
skills/pack/SKILL.md              # serialize a project → bundle
skills/unpack/SKILL.md            # extract a bundle → files
agents/refactorer.md              # shrink & de-dupe, behavior-preserving
agents/reviewer.md                # review diffs against the doctrine
pack.mjs                          # builds BOOTSTRAP.md from these files
BOOTSTRAP.md                      # the self-extracting bundle (generated)
```
