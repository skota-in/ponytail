# Task: apply the token trim (safe, confirm each step)

The audit + measurement are done. The headline: **the GitLab MCP in OpenCode injects 102 tool
schemas (~9,000 tokens) on every prompt — ~55% of the entire always-on budget.** Fix that, do a
little hygiene, and stop. Work carefully; this step **writes** config.

## Rules

1. **Back up before any change** (step 0) — these configs can't be re-downloaded.
2. **Confirm with me before each write.** Show the exact before/after diff; wait for my "yes".
3. **Never touch auth or providers.** Do NOT modify or remove `@statefarm/*` plugins,
   `ghcp`/`bedrock`/`litellm` provider config, or any auth token. Those are model access, not bloat.
4. **Never inline secrets.** `GITLAB_PERSONAL_ACCESS_TOKEN` stays read-from-env.
5. **Verify versioned flags against the package's own docs**, not memory (`--help`/README) before using them.

## Step 0 — Back up (do first, no confirmation needed)

Pack the current config of all four tools into one restorable markdown bundle so every later change
is reversible:
- Read `~/.claude/` (settings.json, CLAUDE.md, skills/, agents/), `~/.codex/` (config.toml,
  AGENTS.md, skills/), `~/.config/opencode/` (opencode.json, AGENTS.md), `~/.copilot/`
  (instructions, mcp config, skills/, agents/).
- Redact every secret (`*token*`, `*key*`, `*secret*`, `Bearer …`, `ghp_…`, etc. → `***REDACTED***`).
- Write them as one self-extracting bundle `~/agent-config-backup.md` using
  `==== BEGIN FILE: <path> ====` / `==== END FILE ====` blocks, so any agent can restore them later.
- Confirm the file exists and report its size. THEN proceed.

## Step 1 — GitLab MCP (the 55% win)

In `~/.config/opencode/opencode.json`, the `gitlab` MCP exposes 102 tools. Reduce it. Ask me which
route, then apply:

- **A — Filter toolsets (recommended):** find the GitLab-MCP package's toolset/scope flag (check its
  `--help` / README — it warned about duplicate tools across `merge_requests`/`branches`, so
  filtering is supported). Enable only the toolsets I confirm I use (likely `merge_requests` +
  `repositories`). Target: ~10–20 tools instead of 102.
- **B — Project-scope it:** remove `gitlab` from the global `opencode.json` and add it to a
  per-project `opencode.json` only in repos where I use GitLab. Saves the full ~9,000 on all other work.
- **C — Disable it:** comment it out; I'll use `git`/`glab` CLI.

After applying, **re-measure**: restart OpenCode (or re-read the system prompt) and count the
GitLab tool schemas now present. Report tools-before/after and tokens-before/after.

## Step 2 — Remove the dead MCP (hygiene)

`dnr-check-docs` is configured in all four tools but **fails to start** (`statefarm-mcp-remote` not
on PATH; Claude's localhost:8080 health-check fails). It costs 0 tokens but clutters config and
hangs startup. Ask me: **fix it** (correct the package path / endpoint) or **remove it** from all
four. Apply my choice.

## Step 3 — One shared, leaner doctrine (optional)

The three identical `AGENTS.md`/`CLAUDE.md` (~833 tok each) are Ponytail copies. If I agree:
- Replace them with the single **Lean-lite doctrine** (I'll provide `BOOTSTRAP.user.lite.md`, or
  reuse my existing one) — ~561 tok, and it adds safety guardrails + a `fresh-docs` rule + a
  readability floor Ponytail lacked. Keep one canonical `~/AGENTS.md`; point `CLAUDE.md` at it with
  `@AGENTS.md`. Don't duplicate rules across files.

## Step 4 — Report

Print a before/after table: always-on tokens per tool, before vs after, and the total saved per
prompt. Confirm the backup path. List anything you intentionally did **not** touch (auth, providers,
worktree hooks, domain skills).

Do not change anything outside steps 1–3, and nothing at all without my go-ahead per step.
