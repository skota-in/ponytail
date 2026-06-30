# Task: inventory my AI coding-agent setup (read-only)

You are auditing how my four AI coding tools are configured on this machine:
**OpenCode, Codex, Claude Code, and GitHub Copilot.** I want a complete, accurate
inventory of every **MCP server, skill, subagent, hook, custom command/prompt, and
instruction file** each tool has — so I can decide what to keep, merge across tools, and
trim to save tokens.

## Hard rules

1. **READ-ONLY.** Do not create, edit, move, or delete any config. Only read files and run
   list/`--help` commands. Make no network calls.
2. **REDACT SECRETS before printing.** Whenever you show a config value, mask anything that
   looks like a credential — keys named `*key*`, `*token*`, `*secret*`, `*password*`,
   `authorization`, and any value matching `sk-…`, `ghp_…`, `github_pat_…`, `xoxb-…`,
   `Bearer …`, or a long hex/base64 blob — replace the value with `***REDACTED***`. I'm going
   to photograph the output, so no secret may appear on screen.
3. **Skip what isn't installed.** If a tool or path is missing, note "not found" and move on.
   Don't guess — only report what you actually find on disk or from a command.

## Where to look (try the list command first, then the files, then glob)

Run these from a shell. Adapt paths if your OS differs; on Windows use the equivalent
`%USERPROFILE%`/`%APPDATA%` locations.

**Claude Code**
- `claude mcp list` · `claude --version`
- `~/.claude/settings.json`, `./.claude/settings.json`, `./.claude/settings.local.json` (hooks, statusline)
- MCP: `~/.claude.json` → its `mcpServers` and each `projects.*.mcpServers` (extract keys only; redact env), and any `./.mcp.json`
- Dirs: `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude/commands/`, `~/.claude/plugins/`, and the `./.claude/` equivalents
- Instructions: `~/.claude/CLAUDE.md`, `./CLAUDE.md`, `./AGENTS.md`

**Codex**
- `codex mcp list` · `codex --version`
- `~/.codex/config.toml` (look for `[mcp_servers.*]`, redact env)
- Dirs: `~/.codex/skills/`, `~/.codex/prompts/`
- Instructions: `~/.codex/AGENTS.md`, `./AGENTS.md`

**OpenCode**
- `~/.config/opencode/opencode.json` / `opencode.jsonc` and `./opencode.json` (look for `mcp`, `agent`, `command`, `instructions`, `plugin` keys; redact)
- Dirs: `~/.config/opencode/agent/`, `~/.config/opencode/command/`, `~/.config/opencode/skills/`, and `./.opencode/**`
- Instructions: `./AGENTS.md`

**GitHub Copilot**
- CLI: `~/.copilot/` — `mcp-config.json` or `config.json` (MCP servers, redact), and `~/.copilot/copilot-instructions.md`
- Editor: `./.vscode/mcp.json`, `./.github/copilot-instructions.md`, `./.github/agents/`, `./.github/skills/`, and any `github.copilot*` keys in the VS Code user `settings.json`

If a tool exposes a list command I didn't name, use it. As a safety net, also run a shallow
glob for anything missed:
`find ~ -maxdepth 4 \( -iname "mcp*.json" -o -iname "*.mcp.json" -o -iname "config.toml" \) 2>/dev/null | grep -Ei 'codex|opencode|copilot|claude'`

## For each MCP server, capture (this is the token-cost driver)

Name · transport (stdio command / http url, redacted) · which tool(s) it's configured in ·
a one-line "what it does" · and **how many tools it exposes** if you can tell (from a list
command, or note "unknown — N tools load into context when active"). MCP tool schemas are
what inflate context, so the tool count per server matters most for trimming.

## Output format (optimize for a phone photo)

Print a compact, well-structured report to the terminal — short tables, no giant file dumps.
Keep each tool to roughly one screen; if something is long, summarize and give counts.

1. **Per tool** (Claude / Codex / OpenCode / Copilot) — four small tables:
   `MCP servers` (name · transport · #tools · purpose) ·
   `Skills` (name · purpose) · `Subagents` (name · purpose) ·
   `Hooks / commands / instructions` (what · where).
2. **Cross-tool overlap** — one table: item · type · which tools have it · duplicate? This is
   what we'll merge.
3. **Token-heavy / trim candidates** — bullet list: the biggest instruction files (with rough
   token size = chars ÷ 4), MCP servers exposing many tools, and anything configured but
   unused or duplicated. Rank biggest-first.
4. **One-line totals** — e.g. "Claude: 5 MCP (≈40 tools), 3 skills, 2 agents, 4 hooks".

Also save the full untruncated inventory (secrets still redacted) to `agent-inventory.md` in
the current directory, as a backup in case the on-screen version is clipped.

Then stop — do not change anything. I'll review the report and tell you what to merge next.
