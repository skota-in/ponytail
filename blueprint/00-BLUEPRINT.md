# Ponytail — Build Blueprint (file 0 of 4)

> _Provenance & scope: this is a build specification for the open-source,
> MIT-licensed **ponytail** project (https://github.com/DietrichGebert/ponytail).
> It instructs a code-generation agent and describes the tool's
> context-injection mechanism in plain terms. It contains no secrets, no network
> calls, and no executable payload._

> **You are an AI coding agent (Codex / Copilot / Claude). Your job: regenerate
> the entire `ponytail` repository from these four markdown files.** Read all
> four first, then build in the order given here. Everything you need is in
> these files — there is no hidden source.

### The four files

| # | File | Contents |
|---|------|----------|
| 0 | `00-BLUEPRINT.md` | **This file.** Overview, the ladder, modes, full file tree, build order, conventions, acceptance criteria. |
| 1 | `01-RULESET-AND-SKILLS.md` | Canonical `AGENTS.md` + the six `SKILL.md` files + the derived rule copies. |
| 2 | `02-RUNTIME-AND-HOOKS.md` | Full source of the Node hook layer, statusline scripts, hook maps. |
| 3 | `03-PACKAGING-AND-SUPPORT.md` | Part A: `package.json`, plugin manifests, OpenCode/pi/MCP adapters, commands, scripts, CI. Part B: README, docs, examples, benchmarks, tests. |

**Reproduce verbatim any text or source shown in a fenced block.** Where a file
is only *described* (translations, some examples), generate faithful content
that satisfies the stated contract. When done, run the acceptance checks in §8.

---

## 1. What you are building

`ponytail` is a **distributable "lazy senior dev" skill for AI coding agents**.
Installed into an agent, it forces the *laziest solution that actually works*:
YAGNI first, then reuse, then the standard library, then native platform
features, then an existing dependency, then one line, and only then new code —
without ever cutting validation, error handling, security, or accessibility.

The trick of the repo: **one ruleset, many hosts.** The behavior is authored
once and shipped to ~16 different agent platforms (Claude Code, Codex, GitHub
Copilot, OpenCode, Gemini/Antigravity CLI, pi, Cursor, Windsurf, Cline, Kiro,
Zed, CodeWhale, Swival, OpenClaw, plus a standalone MCP server). Each platform
gets a *thin adapter* that points back at the same shared text and the same
shared Node.js instruction-builder. **Do not let the ruleset drift between
copies** — a CI script enforces byte-equality.

It is itself a "ponytail" project: small, DRY, boring-on-purpose. When you
regenerate it, obey its own rules.

### The mascot / voice
The README persona is a long-ponytailed senior dev with oval glasses who has
"been at the company longer than the version control": you show him fifty
lines, he says nothing and replaces them with one. Keep that dry voice in
prose (README, skill text). Keep code itself plain.

### Identity / metadata (use verbatim)
- npm package name: `@dietrichgebert/ponytail`
- Author: `Dietrich Gebert`, https://github.com/DietrichGebert
- Repo: `https://github.com/DietrichGebert/ponytail`
- License: **MIT**
- **Version: `4.8.3`** — this exact string appears in `package.json`,
  `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`,
  `gemini-extension.json`, and `ponytail-mcp/package.json`. A CI script
  (`scripts/check-versions.js`) asserts they all match. Keep them in sync.
- Brand color: `#111111`.

---

## 2. The single most important artifact: the ladder

Every piece of prose in this repo is a restatement of one decision procedure.
Memorize it; you will write it (in long and short forms) many times.

```
Before writing any code, stop at the first rung that holds:
1. Does this need to exist at all?        → no: skip it, say so in one line (YAGNI)
2. Already in this codebase?              → reuse the helper/util/pattern, don't rewrite
3. Stdlib does it?                        → use it
4. Native platform feature covers it?     → use it  (<input type="date"> over a lib, CSS over JS, DB constraint over app code)
5. Already-installed dependency solves it?→ use it; never add a new dep for a few lines
6. Can it be one line?                    → one line
7. Only then:                             → the minimum code that works
```

Non-negotiable companions to the ladder (these appear in *every* full copy):
- **The ladder runs *after* you understand the problem, not instead of it.**
  Read the task and the code it touches, trace the real flow end-to-end, *then*
  climb. "A small diff you don't understand is laziness dressed up as
  efficiency."
- **Bug fix = root cause, not symptom.** Grep every caller of the function you
  touch; fix the shared function once (smaller diff than one guard per caller,
  and patching only the named path leaves sibling callers broken).
- **Never lazy about:** understanding the problem, input validation at trust
  boundaries, error handling that prevents data loss, security, accessibility,
  hardware calibration (a real clock drifts, a sensor reads off), anything
  explicitly requested.
- **Mark deliberate shortcuts** with a `ponytail:` comment. If the shortcut has
  a ceiling, the comment names the ceiling *and* the upgrade path, e.g.
  `# ponytail: global lock, per-account locks if throughput matters`.
- **Lazy code without its check is unfinished.** Non-trivial logic leaves ONE
  runnable check behind (an `assert`-based `demo()`/`__main__` self-check or one
  small test file — no frameworks, no fixtures). Trivial one-liners need no test.

Full canonical text lives in **file 1** (`01-RULESET-AND-SKILLS.md`).

---

## 3. Modes / intensity levels

Ponytail has runtime modes. The active mode is tracked in a flag file and shown
in a statusline badge.

| Mode | Meaning |
|------|---------|
| `lite` | Build what's asked, but name the lazier alternative in one line. User picks. |
| `full` | **Default.** The ladder enforced; stdlib & native first; shortest diff and explanation. |
| `ultra` | YAGNI extremist. Deletion before addition. Ship the one-liner and challenge the rest of the requirement in the same breath. |
| `off` | Deactivated — no flag, no rules injected. |
| `review` | Internal mode set by `/ponytail-review`; behavior defined by that skill. |

- **Valid config modes:** `off, lite, full, ultra, review`.
- **Valid runtime modes:** `off, lite, full, ultra`.
- Default mode resolution order: `PONYTAIL_DEFAULT_MODE` env var → config file
  `defaultMode` field → `full`.
- Config file path: `$XDG_CONFIG_HOME/ponytail/config.json`, else
  `~/.config/ponytail/config.json` (mac/Linux), else
  `%APPDATA%\ponytail\config.json` (Windows).
- Deactivation phrases (whole message only, case-insensitive, trailing
  punctuation stripped): `stop ponytail` or `normal mode`.

---

## 4. Commands (six)

Each command exists in several host-specific shapes (TOML for Claude Code,
`.md` for OpenCode, `SKILL.md` for skill hosts). Same six everywhere:

| Command | What it does |
|---------|--------------|
| `/ponytail [lite\|full\|ultra\|off]` | Set intensity (no arg reports current). |
| `/ponytail-review` | Review the current **diff** for over-engineering; hands back a delete-list. |
| `/ponytail-audit` | Audit the whole **repo** for over-engineering. |
| `/ponytail-debt` | Harvest `ponytail:` shortcut comments into a tracked ledger. |
| `/ponytail-gain` | Show the measured-impact scoreboard (less code/cost, more speed). |
| `/ponytail-help` | Quick reference for all of the above. |

In Codex these are invoked with `@` (`@ponytail-review`); in Copilot CLI they
are namespaced (`/ponytail:ponytail-review`).

---

## 5. Complete file tree

Build exactly this tree (138 files). `★` = load-bearing logic given verbatim in
files 1–3. Everything else is derived text or boilerplate you can regenerate
from the specs.

```
.
├── README.md                         # main readme (persona, numbers, install matrix) — file 3 (Part B)
├── README.es.md  README.ko.md        # Spanish + Korean translations of README
├── AGENTS.md                       ★ # CANONICAL compact ruleset (source of truth for copies) — file 1
├── LICENSE                           # MIT, "Dietrich Gebert"
├── package.json                    ★ # npm manifest, version 4.8.3 — file 3
├── opencode.json                     # { plugin: ["./.opencode/plugins/ponytail.mjs"] }
├── gemini-extension.json             # { name, version 4.8.3, contextFileName: "AGENTS.md" }
├── .env.example                      # ANTHROPIC_API_KEY=sk-ant-<your-key-here>
├── .gitignore                        # node_modules, .env, benchmark output, etc.
│
├── skills/                         ★ # SIX canonical SKILL.md files — file 1
│   ├── ponytail/SKILL.md
│   ├── ponytail-review/SKILL.md
│   ├── ponytail-audit/SKILL.md
│   ├── ponytail-debt/SKILL.md
│   ├── ponytail-gain/SKILL.md
│   └── ponytail-help/SKILL.md
│
├── hooks/                          ★ # Node lifecycle hooks + statusline — file 2
│   ├── ponytail-config.js            #   config/mode resolver (shared core)
│   ├── ponytail-runtime.js           #   host detection + flag I/O + hook output
│   ├── ponytail-instructions.js      #   builds ruleset text from SKILL.md, filtered by mode
│   ├── ponytail-activate.js          #   SessionStart hook
│   ├── ponytail-mode-tracker.js      #   UserPromptSubmit hook (mode switching)
│   ├── ponytail-subagent.js          #   SubagentStart hook (inject into subagents)
│   ├── ponytail-statusline.sh        #   POSIX statusline badge
│   ├── ponytail-statusline.ps1       #   Windows statusline badge
│   ├── claude-codex-hooks.json       #   hook map for Claude Code + Codex
│   └── copilot-hooks.json            #   hook map for Copilot CLI
│
├── commands/                         # Claude Code command TOMLs (6) — file 3
│   └── ponytail*.toml
│
├── .claude-plugin/                   # Claude Code plugin — file 3
│   ├── plugin.json
│   └── marketplace.json
├── .codex-plugin/plugin.json         # Codex plugin manifest — file 3
├── .github/
│   ├── plugin/{plugin.json,marketplace.json}  # Copilot CLI plugin (mirror of .claude-plugin)
│   ├── copilot-instructions.md       # instruction-tier copy of AGENTS.md
│   ├── FUNDING.yml
│   └── workflows/{test.yml,publish.yml}        # CI — file 3
│
├── .opencode/                        # OpenCode adapter — file 3
│   ├── plugins/ponytail.mjs        ★ #   server plugin
│   └── command/ponytail*.md          #   6 command files (frontmatter + body)
│
├── pi-extension/                     # pi agent harness extension — file 3
│   ├── index.js                    ★
│   ├── package.json
│   └── test/{extension.test.js,helpers.test.js}
│
├── ponytail-mcp/                     # standalone MCP server — file 3
│   ├── index.js
│   ├── instructions.js
│   ├── package.json
│   ├── README.md
│   └── test/instructions.test.js
│
├── .agents/                          # generic "AGENTS rules" host
│   ├── rules/ponytail.md             #   = AGENTS.md body (copy)
│   └── plugins/marketplace.json
├── .openclaw/skills/                 # OpenClaw skills — GENERATED from skills/ — file 3
│   └── ponytail*/SKILL.md   (×6)
│
│   # ---- instruction-tier rule copies (all derived from AGENTS.md body) ---- file 1
├── .cursor/rules/ponytail.mdc        # frontmatter + AGENTS body
├── .windsurf/rules/ponytail.md       # AGENTS body
├── .clinerules/ponytail.md           # AGENTS body
├── .kiro/steering/ponytail.md        # frontmatter + AGENTS body
│
├── docs/
│   ├── agent-portability.md          # the host→files mapping table — file 3 (Part B)
│   └── platform-native.md            # catalogue of native features the ladder reaches for
│
├── examples/                         # before/after showcases (11 + README) — file 3 (Part B)
│   └── *.md
│
├── benchmarks/                       # promptfoo + agentic benchmark harness — file 3 (Part B)
│   ├── README.md  agentic/  arms/  results/  *.js  *.py  *.yaml  prompts.json
│
├── tests/                            # node --test suite (10 files) — file 3 (Part B)
│   └── *.test.js
│
├── scripts/                          # maintenance scripts — file 3
│   ├── check-rule-copies.js          #   CI: assert rule copies match AGENTS.md
│   ├── check-versions.js             #   CI: assert version strings match
│   ├── build-openclaw-skills.js      #   regenerate .openclaw/skills from skills/
│   ├── publish-openclaw-skills.js
│   └── uninstall.js                  #   clean up state outside plugin dir
│
└── assets/                           # logo.png/.svg, logo-dark.*, social-preview.png,
                                      #   benchmark-3model.svg, benchmark-agentic.svg
```

---

## 6. Build order

1. **Ruleset & skills** (file 1): `AGENTS.md` first (it is the source of
   truth), then the six `skills/*/SKILL.md`, then derive the instruction-tier
   copies (`.cursor`, `.windsurf`, `.clinerules`, `.kiro`, `.agents/rules`,
   `.github/copilot-instructions.md`).
2. **Runtime & hooks** (file 2): the five `.js` hooks + two statusline scripts +
   two hook-map JSONs. This is the real logic — copy it faithfully.
3. **Packaging** (file 3): `package.json`, all plugin manifests, OpenCode
   plugin, pi-extension, MCP server, command files, scripts, CI. Then run
   `node scripts/build-openclaw-skills.js` to generate `.openclaw/skills/`.
4. **Support material** (file 3, Part B): README + translations, docs, examples,
   benchmarks, tests. Stop and run the acceptance checks.

---

## 7. Conventions (apply throughout — they are themselves "ponytail")

- **Node, no build step, near-zero deps.** Hooks are plain CommonJS Node ≥18,
  no `node_modules` required to run. Only `ponytail-mcp` has runtime deps
  (`@modelcontextprotocol/sdk`, `zod`); pi-extension and tests are dependency-free.
- **Shared core, thin adapters.** `hooks/ponytail-config.js` and
  `hooks/ponytail-instructions.js` are the single source of behavior; OpenCode
  (`.mjs`), pi (`index.js`), and the Claude/Codex hooks all `require()` them.
  Never reimplement mode logic in an adapter.
- **Fail silent in hooks.** Every hook wraps its work in try/catch and exits 0.
  A hook must never break a session (missing `node`, closed stdout/EPIPE, bad
  JSON all degrade quietly).
- **Cross-platform.** Strip UTF-8 BOM before `JSON.parse` of files that might be
  Windows-authored. Tolerate CRLF. Provide a `commandWindows`/PowerShell variant
  for every shell hook command.
- **Keep copies aligned.** After touching ruleset text, the body of every
  instruction-tier copy must still equal the `AGENTS.md` body (minus
  frontmatter); `scripts/check-rule-copies.js` enforces it. After touching a
  skill, rerun `build-openclaw-skills.js`.

---

## 8. Acceptance criteria (the build is "done" when)

```bash
node scripts/check-rule-copies.js   # all instruction-tier copies match AGENTS.md
node scripts/check-versions.js      # 4.8.3 everywhere
node scripts/build-openclaw-skills.js && git diff --exit-code .openclaw  # skills not stale
npm test                            # node --test tests/*.test.js  AND  pi-extension tests pass
```

`npm test` runs `node --test tests/*.test.js && npm test --prefix pi-extension`.
The suite covers: hooks behavior (mode flag I/O, host detection, hook output
shapes), Windows host paths, command/skill presence, copilot & opencode &
gemini adapters, openclaw skill staleness, uninstall cleanup, and the
correctness/behavior gates. Build the tests from the contracts described across
files 2–4; each must pass against the source you generate.

> Continue to `01-RULESET-AND-SKILLS.md`.
