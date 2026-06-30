# Task: measure the always-on token cost (read-only)

Follow-up to the inventory. We now know *what's* configured; this measures the **always-on**
cost — the tokens charged on **every prompt** regardless of the task. Read-only, redact secrets,
change nothing.

## Why only these things

Skill *bodies*, subagent prompts, and slash commands load **on demand** — they cost nothing until
triggered, so ignore them here. What's charged on every request is: instruction/memory files,
**MCP server tool schemas**, plugin tool schemas, and skill *descriptions* (picker metadata).
Measure exactly those.

## 1. MCP tool counts (the big unknown)

For each MCP server, get the number of tools it exposes **and** the rough token size of their
schemas (names + descriptions + input schemas are what land in context).

- Claude: `claude mcp list` and, if available, a tools/inspect call per server. Connect and count
  tools for `dnr-check-docs`.
- Codex: list tools for `dnr-check-docs` and `node_repl`.
- OpenCode: list tools for `gitlab` and `dnr-check-docs`.
- Copilot: list tools for `dnr-check-docs`.
- If a tool-list command isn't exposed, start the server's stdio command with the MCP
  `tools/list` request (or read its README/`--help`) to count tools. If a server won't start
  (e.g. the failed `dnr-check-docs` in Claude), record "unreachable — 0 measurable, still costs a
  connection attempt".

For each server report: **server · #tools · est. schema tokens (chars of the tools/list JSON ÷ 4)
· which tools it's configured in.**

## 2. Codex marketplace plugins

`sites, documents, pdf, spreadsheets, presentations, template-creator` may each register tools
that are always-on like MCP. For each, note whether it exposes tools and the count/size if so.

## 3. Skill descriptions (always-on picker text)

For each tool, sum the `description:` frontmatter sizes across all its skills (NOT the bodies):
- Claude/Codex: the 6 skills' descriptions.
- Copilot: all 36 skills' descriptions (this is the always-on slice of that ~85K).
Report total chars ÷ 4 = tokens, per tool.

## 4. Instruction/memory files

Confirm the per-prompt size of each loaded instruction file already found
(`CLAUDE.md`, the three `AGENTS.md`, `copilot-instructions.md`) — chars ÷ 4.

## Output (photo-friendly)

A single ranked table — **"Always-on token cost, per prompt"** — biggest first:

`Source · type (MCP/plugin/skill-desc/instructions) · which tool(s) · est. tokens/prompt`

Then three lines:
1. **Always-on total per tool** (Claude / Codex / OpenCode / Copilot).
2. **The single biggest always-on item** and what removing/consolidating it saves.
3. **Grand total always-on** vs the audit's 110K worst-case — so we see how much is actually
   on-demand.

Save the full numbers to `agent-tokens.md` (secrets redacted). Then stop — change nothing. I'll
turn this into the final trim.
