# Lean Kit — copy-and-paste bundles

Each file below is a **self-extracting markdown "zip"**: copy the whole file, paste it into
**Claude Code** or **Codex**, and tell the agent to unzip it. It rebuilds the Lean doctrine,
skills, and hooks on disk — no clone, no download, no Python.

## Which one to copy

| File | Scope | Token overhead | Paste, then say |
|------|-------|----------------|-----------------|
| **`BOOTSTRAP.lite.md`** | this repo only | ~727 / session | "unzip this bundle into the project" |
| `BOOTSTRAP.md` | this repo only | ~2,150 / session | "unzip this bundle into the project" |
| **`BOOTSTRAP.user.lite.md`** | global — every repo | ~727 / session | "unzip this global bundle into my home config" |
| `BOOTSTRAP.user.md` | global — every repo | ~2,150 / session | "unzip this global bundle into my home config" |

- **`lite`** = compressed doctrine + trimmed skill descriptions + functional-only hooks. Same rules
  and safety guardrails, ~⅓ the always-in-context cost. Recommended.
- **`user`** = installs into your home config (`~/.claude`, `~/.codex`, `~/.copilot`,
  `~/.config/opencode`) so it applies to every repo. The agent confirms `$HOME` first and **merges**
  into an existing `settings.json` (never overwrites it). Omit it for one project.

**Start here:** copy `BOOTSTRAP.lite.md` for a single project, or `BOOTSTRAP.user.lite.md` to apply
it everywhere.

## What you get

A "lazy senior dev" doctrine for your AI agent: climb the ladder (reuse → stdlib → native → one
line) before writing new code, keep it readable, confirm versioned APIs against current docs, and
never cut validation, security, or accessibility — plus `lean` / `fresh-docs` / `review` skills and
optional Claude hooks (auto-format on edit, verify gate on stop).

## Optimize an existing setup (audit → measure → trim)

If you already run skills/MCP/hooks across Claude, Codex, OpenCode, and Copilot, these prompts cut
the token bill. Paste each into Claude Code; all are **read-only and redact secrets** (photo-safe).

| File | What it does |
|------|--------------|
| `prompt.md` | Inventories every MCP server, skill, agent, hook, and instruction file across all four tools. |
| `analysis.md` | The findings + a ranked trim plan (always-on vs on-demand tokens, dedupe, safety flags). Read this. |
| `prompt-measure.md` | Measures the always-on cost that the inventory couldn't — tools-per-MCP-server. Run after `prompt.md`. |

**Key idea:** the scary "100K+ tokens" is mostly skill *bodies*, which load **on demand** — you only
pay for them when a task uses them. The bill you pay *every prompt* is instruction files + **MCP tool
schemas** + skill descriptions. Shrink those; keep the heavy bodies for when you need them. Full
reasoning and the step-by-step plan are in `analysis.md`.

## Compliance

Pure text, Node/sh only — no Python, no network calls, no telemetry, no new dependencies. The only
files that execute anything are the optional hooks in `.claude/settings.json`; review or delete that
file to skip them.
