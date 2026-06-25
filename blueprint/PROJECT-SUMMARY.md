# Ponytail — Project Summary

> _Provenance & scope: overview of the open-source, MIT-licensed **ponytail**
> project. Descriptive prose only; no secrets, no network calls, no executable
> payload._

A plain-English overview of what this project is, whether it's safe to run on a
managed/enterprise laptop, and how to use it with the four agents you care about:
**GitHub Copilot, Claude (Claude Code), OpenCode, and Codex.**

---

## 1. What it is, in one paragraph

Ponytail is a small **"lazy senior dev" behavior pack** for AI coding agents.
Once installed, it sits in the background of your agent and nudges it to write
*the smallest solution that actually works* instead of over-engineering: skip
what isn't needed (YAGNI), reuse what's already there, reach for the standard
library and built-in platform features before adding dependencies, prefer one
line over fifty — **without ever cutting validation, error handling, security,
or accessibility.** It's not a separate app you run; it's a set of instructions
(and a few tiny startup scripts) that your existing agent loads.

Think of it as a house style that says: *"Do less, but never do it carelessly."*

## 2. What it actually does for you

- **Less code per task.** On the project's own benchmark it produced ~54% less
  code on average (up to ~94% where an agent would otherwise over-build, e.g.
  reaching for a date-picker library instead of `<input type="date">`).
- **Cheaper and faster** as a side effect (fewer tokens generated).
- **A few helper commands** you can call on demand:

| Command | What it does |
|---------|--------------|
| `/ponytail [lite\|full\|ultra\|off]` | Turn it on/off or set how aggressive it is (default `full`). |
| `/ponytail-review` | Review your current diff and list what to delete/simplify. |
| `/ponytail-audit` | Same, but across the whole repo. |
| `/ponytail-debt` | Collect every `ponytail:` "I took a shortcut here" comment into one ledger. |
| `/ponytail-gain` | Show the measured impact scoreboard. |
| `/ponytail-help` | Quick reference. |

**The decision rule it applies (the "ladder"):**
```
1. Does this need to exist at all?    → no: skip it (YAGNI)
2. Already in this codebase?          → reuse it
3. Standard library does it?          → use it
4. Native platform feature covers it? → use it
5. An installed dependency solves it? → use it
6. Can it be one line?                → one line
7. Only then: the minimum that works
```
It runs this *after* reading and understanding the problem — it makes the
solution smaller, never the comprehension.

---

## 3. Enterprise / compliance notes (read this part)

This is the section that matters for a managed laptop. The facts below were
verified against the source in this repo.

### ✅ Runs on Node.js — no Python needed to install or use it
- The entire **runtime path is plain Node.js** (CommonJS, Node ≥ 18). The
  startup scripts in `hooks/` and the adapters for your four agents are all
  JavaScript.
- **Python is only used in the optional `benchmarks/` folder** — the
  measurement harness the authors used to produce their numbers. You do **not**
  need Python to install, run, or use ponytail. **You can delete the
  `benchmarks/` folder entirely** and everything still works. (The example
  `.md` files happen to *show* Python code samples as illustrations, but that's
  just sample text, not something that executes.)
- If you ever run `npm test` at the repo root, note that the correctness tests
  spawn Python. **Skip that** — it's for the project's own CI, not for using the
  skill. You never need to run it.

### ✅ No network calls, no telemetry in the runtime
- The startup hooks and agent adapters do **only local file reads/writes**
  (a small flag file recording the active mode) and print text back to your
  agent. A grep of the runtime code (`hooks/`, the OpenCode plugin, the pi
  adapter) found **zero** HTTP/fetch/socket calls. Nothing phones home.
- The only place a network call happens anywhere in the repo is the optional
  benchmark harness, which calls the Anthropic API — again, not part of using
  the skill, and removable.

### ✅ Zero third-party dependencies in the core
- The hooks and skills have **no npm dependencies** — pure Node standard
  library. Nothing is downloaded at runtime.
- The *only* component with npm dependencies is the optional standalone
  **MCP server** (`ponytail-mcp/`, which uses `@modelcontextprotocol/sdk` and
  `zod`). You don't need it for Copilot/Claude/OpenCode/Codex, so you can ignore
  that folder too.

### ✅ License
- **MIT.** Permissive, enterprise-friendly. One file, `LICENSE`.

### What actually executes on your machine
When active, ponytail runs two tiny Node scripts at the lifecycle points your
agent defines:
1. **On session start** — reads your configured mode and hands the agent the
   ruleset as hidden context; writes a one-line flag file (e.g. `~/.claude/.ponytail-active`).
2. **On each prompt you submit** — checks if you typed a `/ponytail` command and
   updates that flag file.

Every script is wrapped to **fail silently and never block your session** — if
`node` isn't on your PATH, the hooks just stay quiet and the plain-text rules
still work. Nothing requires admin rights.

> **Bottom line for IT review:** MIT-licensed, Node-only, no dependencies in the
> core, no network access, no telemetry, all state is local files in your own
> home directory. The only non-Node, network-touching parts live in
> `benchmarks/` and `ponytail-mcp/`, both optional and safely deletable.

---

## 4. Using it with your four agents

You only need the parts below. Everything else in the repo (other agents'
adapters, benchmarks, MCP server) can be ignored.

### Claude Code
Install from the marketplace (send as two separate prompts):
```
/plugin marketplace add DietrichGebert/ponytail
```
```
/plugin install ponytail@ponytail
```
It activates every session at the `full` level. Switch with `/ponytail lite`,
`/ponytail ultra`, or `/ponytail off`. A `[PONYTAIL]` badge can show in your
status line. Requires `node` on your PATH for the always-on activation (the
skill text still works without it).

### Codex
```bash
codex plugin marketplace add DietrichGebert/ponytail
codex
```
Then open `/plugins`, install Ponytail, open `/hooks`, review and trust its two
Node lifecycle hooks, and start a new thread. Commands are invoked with `@`,
e.g. `@ponytail-review`. Also works in the Codex desktop app and the VS Code
Codex extension (which reads `AGENTS.md` from the repo root).

### GitHub Copilot CLI
```bash
copilot plugin marketplace add DietrichGebert/ponytail
copilot plugin install ponytail@ponytail
```
In an interactive session the slash forms also work
(`/plugin marketplace add …`, `/plugin install …`). Commands are namespaced,
e.g. `/ponytail:ponytail-review`.

**GitHub Copilot (editor / instruction-only):** if you can't or don't want to
install a plugin, just copy **`.github/copilot-instructions.md`** into your
repo (or `~/.copilot/copilot-instructions.md` for all repos). That gives you the
always-on rules with no scripts at all — the most locked-down-friendly option.

### OpenCode
Add to your `opencode.json`:
```json
{ "plugin": ["@dietrichgebert/ponytail"] }
```
It injects the ruleset into every turn at the active level and adds the
`/ponytail` and `/ponytail-review` commands. OpenCode also auto-loads the repo's
`AGENTS.md`, so the rules apply even without the plugin.

### The lowest-friction option for all four
Every one of these agents will read a plain instructions file. If plugins are
restricted on your laptop, you can skip all scripts and just drop the
**`AGENTS.md`** file (Codex, OpenCode) or **`.github/copilot-instructions.md`**
(Copilot) into your project root. No Node, no hooks, no install — just text the
agent reads. You lose the `/ponytail lite|ultra` mode switches and the status
badge, but the core "lazy senior dev" behavior is fully there.

---

## 5. Quick reference: which files you need vs. can ignore

| You need | For |
|----------|-----|
| `AGENTS.md` | Codex, OpenCode, Copilot CLI fallback, any agent — the core rules |
| `skills/` | The six commands (Claude Code, Codex, OpenCode) |
| `hooks/` (Node only) | Always-on activation + mode switching + status badge |
| `.claude-plugin/`, `.codex-plugin/`, `.github/plugin/`, `.opencode/` | Plugin manifests for your four hosts |
| `.github/copilot-instructions.md` | Copilot editor / instruction-only mode |

| You can ignore / delete | Why |
|-------------------------|-----|
| `benchmarks/` | Python + API measurement harness — optional, not needed to use it |
| `ponytail-mcp/` | Standalone MCP server with npm deps — not needed for your four hosts |
| `.cursor/`, `.windsurf/`, `.clinerules/`, `.kiro/`, `pi-extension/`, `gemini-extension.json` | Adapters for other agents you don't use |

---

*Want to rebuild this whole project from scratch with an agent? The four
`blueprint/0X-*.md` files in this folder are a complete build spec for exactly
that.*
