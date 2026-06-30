# Task: reset my agent setup to a lean baseline (back up first, keep auth)

I want to **strip my AI coding tools (Claude Code, Codex, OpenCode, Copilot) back to a clean,
minimal baseline** and add things back only when I actually need them. I added most of this config
just because it existed, not because I use it. Back everything up so removal is reversible, then
remove the discretionary stuff — but keep what gives me model access.

## Golden rules

1. **Back up first (step 0). The backup IS my "add it back later" library.** I can't re-download
   these, so nothing gets deleted until it's in the backup bundle.
2. **NEVER remove auth or model access.** Keep: `@statefarm/opencode-ghcp-auth`, the `ghcp` /
   `amazon-bedrock` / `litellm` provider config, Codex's sandbox + trusted-paths, and each tool's
   model-catalog/login. If you're unsure whether something is auth, **keep it and ask.**
2. **Confirm before deleting.** Show me the full keep-list vs remove-list and wait for my "yes".
3. **Never inline or print secrets.**

## Step 0 — Back up to a restorable bundle (do first)

Pack the **current** config of all four tools into one self-extracting markdown file
`~/agent-config-backup.md`:
- Include: `~/.claude/` (settings.json, CLAUDE.md, skills/, agents/, the worktree hooks),
  `~/.codex/` (config.toml, AGENTS.md, skills/), `~/.config/opencode/` (opencode.json, AGENTS.md),
  `~/.copilot/` (instructions, mcp config, skills/, agents/).
- One clearly-labeled `==== BEGIN FILE: <path> ==== … ==== END FILE ====` block per item, so I can
  restore any single skill/MCP/agent later by copying just its block back.
- **Keep values verbatim** (this is a restore file, not a photo) — but this file then contains real
  config: mark it `LOCAL ONLY — DO NOT SHARE / COMMIT / PHOTOGRAPH`, and don't print its contents.
- Confirm the file exists, report its size and how many blocks it holds. THEN continue.

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
3. List exactly what was removed, where its backup block is, and the one-line way to restore it.

## How I add things back later

When I actually need something, restore its block from `~/agent-config-backup.md` into the right
tool — nothing is lost, it's just not loaded until it earns its place.

Do nothing outside these steps, and nothing at all without my go-ahead at step 1.
