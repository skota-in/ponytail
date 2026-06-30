# Task: reset my agent setup to a lean baseline (back up first, keep auth)

I want to **strip my AI coding tools (Claude Code, Codex, OpenCode, Copilot) back to a clean,
minimal baseline** and add things back only when I actually need them. I added most of this config
just because it existed, not because I use it. Back everything up so removal is reversible, then
remove the discretionary stuff — but keep what gives me model access.

## Golden rules

1. **No full backup needed — I restore things by re-running their install command.** So instead of
   a backup bundle, just keep a one-line **removal manifest** (step 0) of what you took out and
   where it came from, so I know what to re-run later.
2. **NEVER remove auth or model access.** Keep: `@statefarm/opencode-ghcp-auth`, the `ghcp` /
   `amazon-bedrock` / `litellm` provider config, Codex's sandbox + trusted-paths, and each tool's
   model-catalog/login. If you're unsure whether something is auth, **keep it and ask.**
3. **Confirm before deleting.** Show me the full keep-list vs remove-list and wait for my "yes".
4. **Never inline or print secrets.**

## Step 0 — Removal manifest (lightweight, no backup bundle)

As you remove things, record a simple list at `~/agent-removed.md` — one line per removed item:
**name · type (MCP/skill/agent/hook) · which tool · source (npm package / marketplace id / repo /
local file)**. That's my re-add cheat-sheet.

⚠️ One check before deleting: if any item is a **hand-authored local file** (not installable from a
marketplace/package/repo), a reinstall command won't bring it back — flag those and ask me before
removing, in case I want to copy them somewhere first.

## Step 1 — Show the plan, get my OK

Print two lists and stop for confirmation:

- **KEEP (infrastructure):** auth plugin(s), providers, model catalog/login, Codex sandbox +
  trusted paths, and — if I want one always-on doctrine — a single lean instruction file (see step 3).
- **REMOVE (discretionary → lives in the backup):** every MCP server (`gitlab`, `dnr-check-docs`,
  `node_repl`), every skill (Claude 6, Codex 6, Copilot 36), every subagent (`leanix`,
  `servicenow`), the custom worktree hooks, and any duplicate/extra instruction files. Codex's
  desktop marketplace plugins (sites/docs/pdf/…) cost 0 prompt tokens — list them separately and
  ask whether to keep or remove.

## Step 2 — Remove the discretionary config

After my OK, remove the REMOVE-list from each tool's live config (the backup already holds it).
Leave the KEEP-list completely untouched. Make the smallest edits that disable each item; don't
rewrite files wholesale.

## Step 3 — One minimal doctrine (recommended, optional)

Instead of the four duplicated instruction files, install **one** small shared doctrine so the
models still get good habits at near-zero cost. Use my Lean-lite doctrine (~560 tokens: understand
first → climb the ladder → readable → confirm versioned APIs against current docs → never cut
validation/security/accessibility). Keep one canonical `~/AGENTS.md`; point `CLAUDE.md` at it with
`@AGENTS.md`; reuse it for Codex/OpenCode/Copilot. Ask me if I'd rather go fully bare (no doctrine).

## Step 4 — Verify and report

1. **Prove model access survived:** launch each tool and confirm it authenticates / can reach a
   model. If any tool fails to auth, STOP and tell me what changed (we restore from the backup).
2. Print a before/after table: always-on tokens per prompt, per tool, before vs after, and total
   saved. (Baseline before ≈ 15–16.5K across all four; the GitLab MCP alone was ~9K.)
3. Confirm `~/agent-removed.md` lists every removed item with its source, and call out any
   hand-authored local files you preserved.

## How I add things back later

When I actually need something, I re-run its install command (from `~/agent-removed.md`) — or for a
local-file item, copy it back. Nothing is loaded until it earns its place.

Do nothing outside these steps, and nothing at all without my go-ahead at step 1.
