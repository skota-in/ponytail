# Ponytail — Packaging, Adapters, Docs, Examples & Tests (file 3 of 4)

> _Provenance & scope: build specification for the open-source, MIT-licensed
> **ponytail** project. Reproduces manifests and adapter source for a
> code-generation agent. The only `sk-ant-...`-style string is a placeholder in
> `.env.example`; no real secrets, no network calls, no executable payload._

Manifests, the OpenCode/pi/MCP adapters, command files, maintenance scripts,
and CI. Keep the version `4.8.3` consistent everywhere it appears.

---

## 1. `package.json` (repo root)

```json
{
  "name": "@dietrichgebert/ponytail",
  "version": "4.8.3",
  "description": "Lazy senior dev mode for AI agents. The best code is the code you never wrote.",
  "keywords": ["opencode-plugin", "opencode", "ponytail", "pi-package", "pi", "skills"],
  "license": "MIT",
  "author": {
    "name": "Dietrich Gebert",
    "url": "https://github.com/DietrichGebert"
  },
  "homepage": "https://github.com/DietrichGebert/ponytail",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/DietrichGebert/ponytail.git"
  },
  "bugs": {
    "url": "https://github.com/DietrichGebert/ponytail/issues"
  },
  "main": "./.opencode/plugins/ponytail.mjs",
  "exports": {
    ".": "./.opencode/plugins/ponytail.mjs",
    "./plugin": "./.opencode/plugins/ponytail.mjs"
  },
  "files": [
    "AGENTS.md",
    "hooks/",
    "skills/",
    ".opencode/",
    "pi-extension/",
    "assets/",
    "LICENSE"
  ],
  "scripts": {
    "test": "node --test tests/*.test.js && npm test --prefix pi-extension"
  },
  "pi": {
    "extensions": ["./pi-extension/index.js"],
    "skills": ["./skills"]
  },
  "publishConfig": {
    "access": "public"
  }
}
```

---

## 2. Plugin manifests

### `.claude-plugin/plugin.json`
```json
{
  "name": "ponytail",
  "version": "4.8.3",
  "description": "Lazy senior dev mode. Forces the simplest, shortest solution that actually works: YAGNI, stdlib first, no unrequested abstractions.",
  "author": { "name": "Dietrich Gebert", "url": "https://github.com/DietrichGebert" },
  "hooks": "./hooks/claude-codex-hooks.json"
}
```

### `.claude-plugin/marketplace.json`
```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "ponytail",
  "description": "Lazy senior dev mode for AI agents. The best code is the code you never wrote.",
  "owner": { "name": "Dietrich Gebert", "url": "https://github.com/DietrichGebert" },
  "plugins": [
    {
      "name": "ponytail",
      "description": "Forces the laziest solution that works. YAGNI, stdlib first, one line over fifty.",
      "source": "./",
      "category": "productivity"
    }
  ]
}
```

### `.github/plugin/{plugin.json,marketplace.json}` (Copilot CLI)
Mirror of the two files above (Copilot CLI installs via the same
`marketplace add` + `install` flow). The `plugin.json` may point `hooks` at
`./hooks/copilot-hooks.json` for the Copilot-shaped hook map.

### `.agents/plugins/marketplace.json`
A generic-host marketplace manifest, same shape as `.claude-plugin/marketplace.json`.

### `.codex-plugin/plugin.json`
Richer — Codex shows an interface card.
```json
{
  "name": "ponytail",
  "version": "4.8.3",
  "description": "Lazy senior dev mode. Forces the simplest, shortest solution that actually works: YAGNI, stdlib first, no unrequested abstractions.",
  "author": { "name": "Dietrich Gebert", "url": "https://github.com/DietrichGebert" },
  "homepage": "https://github.com/DietrichGebert/ponytail",
  "repository": "https://github.com/DietrichGebert/ponytail",
  "license": "MIT",
  "keywords": ["yagni", "minimalism", "code-review", "productivity"],
  "skills": "./skills/",
  "hooks": "./hooks/claude-codex-hooks.json",
  "interface": {
    "displayName": "Ponytail",
    "shortDescription": "Lazy senior developer mode",
    "longDescription": "Prefer YAGNI, the standard library, native platform features, and the smallest correct implementation.",
    "developerName": "Dietrich Gebert",
    "category": "Productivity",
    "capabilities": ["Instructions", "Lifecycle hooks"],
    "websiteURL": "https://github.com/DietrichGebert/ponytail",
    "defaultPrompt": [
      "Use Ponytail mode for this task.",
      "Review this diff for over-engineering.",
      "Find the smallest correct implementation."
    ],
    "brandColor": "#111111",
    "composerIcon": "./assets/logo.png",
    "logo": "./assets/logo.png"
  }
}
```

### Root single-file manifests
- `gemini-extension.json`:
  `{ "name": "ponytail", "version": "4.8.3", "description": "Lazy senior dev mode. Forces the simplest, shortest solution that actually works: YAGNI, stdlib first, no unrequested abstractions.", "contextFileName": "AGENTS.md" }`
- `opencode.json`:
  `{ "$schema": "https://opencode.ai/config.json", "plugin": ["./.opencode/plugins/ponytail.mjs"] }`
- `.env.example`: two lines — a comment ("Copy to .env (gitignored)…") and
  `ANTHROPIC_API_KEY=sk-ant-<your-key-here>` (a placeholder; no real key)

---

## 3. Command files (six commands × three formats)

### `commands/*.toml` (Claude Code) — one per command
Each TOML has `description` and `prompt`. `commands/ponytail.toml`:
```toml
description = "Switch ponytail intensity level (lite/full/ultra/off)"
prompt = "Switch to ponytail {{args}} mode. If no level specified, use full. Lazy senior dev mode, before any code: does it need to exist at all (YAGNI)? Does the standard library do it? A native platform feature? Can it be one line? Build the minimum that works. No unrequested abstractions, no avoidable dependencies, no boilerplate. Mark intentional simplifications with a ponytail: comment."
```
The other five (`ponytail-review/-audit/-debt/-gain/-help.toml`) each carry a
`description` and a `prompt` that restates the corresponding skill's job (e.g.
review = "Review the current diff for over-engineering; one line per finding,
tag + location + replacement; end with net lines removable").

### `.opencode/command/*.md` (OpenCode) — one per command
Frontmatter (`description:`) + body; OpenCode substitutes `$ARGUMENTS`.
`.opencode/command/ponytail.md`:
```markdown
---
description: Switch ponytail intensity level (lite/full/ultra/off)
---

Switch to ponytail $ARGUMENTS mode. If no level specified, use full. Lazy senior dev mode, before any code: does it need to exist at all (YAGNI)? Does the standard library do it? A native platform feature? Can it be one line? Build the minimum that works. No unrequested abstractions, no avoidable dependencies, no boilerplate. Mark intentional simplifications with a ponytail: comment.
```
The five others mirror their TOML counterparts' prompts.

### Skill format (skill-capable hosts)
The `skills/*/SKILL.md` from file 1 *are* the commands on Claude Code / Codex /
OpenCode / pi / Swival. No extra files needed.

---

## 4. OpenCode adapter — `.opencode/plugins/ponytail.mjs`

ES-module server plugin. Bridges to the CommonJS shared core via
`createRequire`. Keeps its own flag file beside OpenCode's config
(`$XDG_CONFIG_HOME/opencode/.ponytail-active`, else `~/.config/opencode/…`).
Three plugin hooks: `config` (register the six command `.md` files + the
`skills/` dir), `experimental.chat.system.transform` (append the mode-filtered
ruleset every turn unless `off`), and `command.execute.before` (persist
`/ponytail <level>` for the next turn). Reproduce verbatim:

```js
// ponytail — OpenCode plugin.
//
// Injects the ponytail ruleset into every chat's system prompt at the active
// intensity, persists /ponytail mode switches, and registers slash commands so
// they work when the package is installed from npm. Reuses the shared
// instruction builder so Claude Code, Codex, pi, and OpenCode all read one
// source of truth.
//
// OpenCode loads this as a server plugin — add it to your opencode.json:
//   { "plugin": ["@dietrichgebert/ponytail"] }

import { createRequire } from 'module';
import fs from 'fs';
import os from 'os';
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

// The shared instruction builder is CommonJS; bridge to it from this ES module.
const require = createRequire(import.meta.url);
const { getPonytailInstructions } = require('../../hooks/ponytail-instructions');
const { getDefaultMode, normalizePersistedMode } = require('../../hooks/ponytail-config');

// OpenCode has no flag-file convention of its own; keep mode beside its config.
const statePath = path.join(
  process.env.XDG_CONFIG_HOME || path.join(os.homedir(), '.config'),
  'opencode',
  '.ponytail-active',
);

function readMode() {
  try {
    return normalizePersistedMode(fs.readFileSync(statePath, 'utf8').trim()) || getDefaultMode();
  } catch (e) {
    return getDefaultMode();
  }
}

function writeMode(mode) {
  fs.mkdirSync(path.dirname(statePath), { recursive: true });
  fs.writeFileSync(statePath, mode);
}

export function parseCommandFile(filePath) {
  const content = fs.readFileSync(filePath, 'utf8');
  // Tolerate CRLF: a Windows checkout (autocrlf) delivers \r\n, npm ships \n.
  const match = content.match(/^---\r?\n([\s\S]*?)\r?\n---\r?\n([\s\S]*)$/);
  if (!match) return null;
  const description = match[1].match(/description:\s*(.+)/)?.[1]?.trim();
  return { description, template: match[2].trim() };
}

export default async ({ client } = {}) => {
  const log = (level, message) => {
    try { client && client.app && client.app.log({ body: { service: 'ponytail', level, message } }); } catch (e) {}
  };

  const ponytailSkillsDir = path.resolve(__dirname, '../../skills');

  return {
    // Register slash commands + skills directory.
    config: async (config) => {
      if (!config.command) config.command = {};
      const commandDir = path.join(__dirname, '..', 'command');
      try {
        for (const file of fs.readdirSync(commandDir).filter((f) => f.endsWith('.md'))) {
          const name = path.basename(file, '.md');
          const parsed = parseCommandFile(path.join(commandDir, file));
          if (parsed) config.command[name] = parsed;
        }
      } catch (e) {}

      config.skills = config.skills || {};
      config.skills.paths = config.skills.paths || [];
      if (!config.skills.paths.includes(ponytailSkillsDir)) {
        config.skills.paths.push(ponytailSkillsDir);
      }
    },

    // Append the ruleset to the system prompt every turn.
    'experimental.chat.system.transform': async (_input, output) => {
      const mode = readMode();
      if (mode === 'off') return;
      output.system.push(getPonytailInstructions(mode));
    },

    // Persist `/ponytail <level>` so the next turn's injection follows it.
    // ponytail: mode applies from the next message, not the current one — the
    // transform reads the flag the command writes. Good enough; switch to a
    // synchronous store if same-turn switching ever matters.
    'command.execute.before': async (input) => {
      if (!input || input.command !== 'ponytail') return;
      // `off` is persisted like any mode; the transform reads it and stays silent.
      const mode = normalizePersistedMode((input.arguments || '').trim()) || getDefaultMode();
      writeMode(mode);
      log('info', 'ponytail ' + mode);
    },
  };
};
```

---

## 5. pi adapter — `pi-extension/`

`pi-extension/package.json`:
```json
{
  "name": "ponytail-pi-extension-dev",
  "private": true,
  "type": "module",
  "scripts": { "test": "node --test ./test/*.test.js" }
}
```

`pi-extension/index.js` is an ES module that `createRequire`-bridges to the
shared core and exports several pure helpers (used by its tests) plus a default
extension function. Key behaviors:
- Re-exports `filterSkillBodyForMode`; `readDefaultMode = getDefaultMode`.
- `resolveSessionMode(entries, fallback)` — walks session entries backwards for
  the last `customType: "ponytail-mode"` custom entry; returns its mode or the
  fallback.
- `parsePonytailCommand(text, defaultMode)` — returns one of
  `{type:"status"}`, `{type:"set-default",mode}`, `{type:"set-mode",mode}`, or
  `{type:"invalid",reason,…}`. Empty text → set-mode to default (or `full` if
  default is `off`). `"status"` → status. `"default <mode>"` →
  set-default (validated via `normalizeConfigMode`).
- Default export `ponytailExtension(pi)` registers six commands
  (`ponytail`, `ponytail-review/-audit/-gain/-debt/-help`), drives a status bar
  (icons 🌿/⚡/🔥 for lite/full/ultra, `●`/`○` active indicator, 🐴 label),
  appends a `ponytail-mode` session entry on switch, listens to `input` for
  deactivation phrases, resolves mode on `session_start`, toggles the active
  dot on `agent_start`/`agent_end`, and on `before_agent_start` returns
  `{ systemPrompt: existing + "\n\n" + getPonytailInstructions(currentMode) }`
  unless mode is `off`. The five non-`ponytail` commands forward to
  `/skill:ponytail-<name>` via `pi.sendUserMessage` (queued as follow-up if the
  agent is busy).

`pi-extension/test/` has `extension.test.js` and `helpers.test.js` exercising
`parsePonytailCommand`, `resolveSessionMode`, and `filterSkillBodyForMode`.

---

## 6. MCP server — `ponytail-mcp/`

A standalone stdio MCP server for hosts whose only injection point is the prompt
menu. Has its own `package.json` (deps `@modelcontextprotocol/sdk ^1.26.0`,
`zod ^3.23.0`; `type: module`; `private: true`; version `4.8.3`).

- `ponytail-mcp/instructions.js` exports `MODES` (`['lite','full','ultra']`),
  `resolveMode(mode)` (falls back to `PONYTAIL_DEFAULT_MODE`/`full`), and
  `buildInstructions(mode)` (the ruleset text for that intensity — may reuse the
  same compact body as the fallback in file 2).
- `ponytail-mcp/index.js` builds an `McpServer({name:"ponytail",version:"0.1.0"})`,
  registers a **prompt** `ponytail` and a **tool** `ponytail_instructions`, both
  taking an optional `mode` enum arg, both returning `buildInstructions(mode)`.
  The tool is `readOnlyHint: true, openWorldHint: false` with an output schema
  `{ mode: string, instructions: string }`. Connects over `StdioServerTransport`.
- `ponytail-mcp/test/instructions.test.js` checks each mode builds non-empty,
  distinct text and that `resolveMode` honors the env default.
- `ponytail-mcp/README.md` documents running it (`node index.js`) and wiring it
  into an MCP client config.

---

## 7. Maintenance scripts — `scripts/`

All plain Node CommonJS, no deps.

- **`check-rule-copies.js`** (CI gate): reads `AGENTS.md`, strips the trailing
  `(Yes, this file also applies…)` parenthetical to get the *canonical* body,
  then for each copy
  (`.cursor/rules/ponytail.mdc` [strip frontmatter], `.windsurf/rules/ponytail.md`,
  `.clinerules/ponytail.md`, `.agents/rules/ponytail.md`,
  `.github/copilot-instructions.md` [all trim], `.kiro/steering/ponytail.md`
  [strip frontmatter]) asserts the normalized text equals the canonical body;
  `console.error` + non-zero exit on drift. Also asserts a few load-bearing
  **canary** sentences survive verbatim in both `skills/ponytail/SKILL.md` and
  `AGENTS.md` (SKILL.md is longer, so it's a canary, not full equality).
  Normalize line endings (`\r\n`→`\n`) and trim before comparing.

  > ⚠️ **Pick canaries that are contiguous on a single line in BOTH files.**
  > `skills/ponytail/SKILL.md` is hard-wrapped at ~76 cols, so phrases like
  > "The best code is the code never written." are split across two lines there
  > (`The best\ncode is the code never written.`) while being one line in
  > `AGENTS.md` — a naïve substring check fails. Use these three, which are
  > verified contiguous in both: `You are a lazy senior developer`,
  > `Bug fix = root cause, not symptom`, and
  > `input validation at trust boundaries`. (Match the substring after
  > collapsing each file's internal newlines to spaces if you want to be
  > wrap-proof.)

- **`check-versions.js`** (CI gate): asserts the `version`/string `4.8.3`
  matches across `package.json`, `.claude-plugin/plugin.json`,
  `.codex-plugin/plugin.json`, `gemini-extension.json`, `ponytail-mcp/package.json`
  (and any other manifest carrying a version). Exit non-zero on mismatch.

- **`build-openclaw-skills.js`**: regenerates `.openclaw/skills/<name>/SKILL.md`
  for the six skills. Body is copied **verbatim** from `skills/<name>/SKILL.md`
  (so the ruleset never drifts); only the frontmatter is rewritten with a short
  single-line `description` (<160 chars) plus homepage. The short descriptions:
  ```js
  const DESCRIPTIONS = {
    'ponytail': 'Lazy senior dev mode. Forces the simplest, shortest solution that works: YAGNI, stdlib first, no unrequested abstractions.',
    'ponytail-review': 'Review a diff for over-engineering. Finds what to delete: reinvented stdlib, needless deps, speculative abstractions. One line per finding.',
    'ponytail-audit': 'Audit the whole repo for over-engineering. A ranked list of what to delete, simplify, or replace with stdlib or native features.',
    'ponytail-debt': 'Harvest every ponytail: shortcut comment into one debt ledger, so deferrals get tracked instead of forgotten. One-shot report.',
    'ponytail-gain': 'Show ponytail measured impact as a scoreboard: less code, less cost, more speed, from the benchmark medians. One-shot display.',
    'ponytail-help': "Quick reference for ponytail's modes, skills, and commands. One-shot display.",
  };
  const HOMEPAGE = 'https://github.com/DietrichGebert/ponytail';
  ```
  `tests/openclaw-skills.test.js` fails if the committed copies are stale, so run
  this after any skill edit and commit the result.

- **`publish-openclaw-skills.js`**: publishes all six skills to ClawHub at the
  `package.json` version (`clawhub login` once first; `--dry-run` previews).

- **`uninstall.js`**: removes state ponytail writes *outside* the plugin dir —
  the mode flag (`<claudeDir>/.ponytail-active`), the config file
  (`getConfigPath()`), and the `statusLine` entry in `~/.claude/settings.json`
  **only if** it points at ponytail's own script. `removeIfExists` ignores
  `ENOENT`, rethrows other errors.

---

## 8. CI — `.github/workflows/`

### `test.yml`
```yaml
name: test
on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Install Python deps for correctness checks
        run: pip install pandas
      - name: Check rule copies
        run: node scripts/check-rule-copies.js
      - name: Check version consistency
        run: node scripts/check-versions.js
      - name: Run tests
        run: npm test
```

### `publish.yml`
Publishes to npm on `v*` tags (and `workflow_dispatch`) via OIDC trusted
publishing — `permissions: { id-token: write, contents: read }`, Node 22,
`npm install -g npm@latest` (needs npm ≥ 11.5.1 for OIDC), then `npm publish`
(no token; `access` from `publishConfig`).

`.github/FUNDING.yml` is a normal sponsor config.



---

# Part B — Docs, Examples, Benchmarks & Tests

The supporting material: the README that sells it, the docs that map it, the
examples that prove it, the benchmark harness that measures it, and the test
suite that guards it. Build these last, then run the acceptance checks from
file 0 §8.

---

## 1. README.md (repo root)

Tone: dry, deadpan, the long-ponytailed senior dev's voice. Structure:

1. **Logo** (`<picture>` with dark/light `assets/logo*.png`, width 220) +
   centered `<h1>Ponytail</h1>` + tagline *"He says nothing. He writes one
   line. It works."*
2. **Badge row** (shields.io, all color `111111`): stars, release, npm
   (`@dietrichgebert/ponytail`), "works with 14 agents", MIT. Trendshift badges.
3. **One-line results banner:** *"~54% less code (up to 94%) · ~20% cheaper ·
   ~27% faster · 100% safe"* with a `<sub>` footnote explaining the measurement
   (real Claude Code session editing a FastAPI+React repo, mean across 12
   feature tasks, Haiku 4.5, n=4) and linking the full writeup.
4. **Language links:** `README.es.md` (Español) · `README.ko.md` (한국어).
5. **The pitch:** "You know him. Long ponytail. Oval glasses. Has been at the
   company longer than the version control… he replaces fifty lines with one."
6. **Before/after:** the date-picker story → `<input type="date">` with a
   `<!-- ponytail: browser has one -->` comment. Link to `examples/`.
7. **Numbers:** the agentic benchmark table:
   | vs no-skill baseline | LOC | tokens | cost | time | safe |
   | ponytail | −54% | −22% | −20% | −27% | 100% |
   | caveman (terse-prose control) | −20% | +7% | +3% | +2% | 100% |
   | "YAGNI + one-liners" prompt | −33% | −14% | −21% | −30% | 95% |
   Note ponytail is the only arm that cuts every metric *and* stays fully safe.
   A `<details>` holds the older single-shot 80–94% numbers with the caveat that
   the bare-model baseline pads with prose. Image: `assets/benchmark-agentic.svg`.
8. **"The rule was never 'fewest tokens.'"** — it's: write only what the task
   needs, never cut validation/error-handling/security/accessibility.
9. **How it works:** the 7-rung ladder in a code block; "lazy about the
   solution, never about reading."
10. **Install:** a section per host. Note the two-prompt Claude Code install,
    the Codex `/plugins`+`/hooks` flow, Copilot CLI namespacing, pi, OpenCode
    (`opencode.json` snippet), Gemini/Antigravity, CodeWhale, Swival, OpenClaw.
    `node` must be on the non-interactive PATH for the hooks (skills still work
    without it). `PONYTAIL_DEFAULT_MODE` / `config.json` set the default level.
11. **Uninstall table** + the note that `node scripts/uninstall.js` cleans the
    out-of-plugin state, and to run it *before* the host remove command.
12. **Commands table** (the six). 13. **Development** (check-rule-copies, npm
    test, build-openclaw-skills). 14. **FAQ** (dry one-liners). 15. **License:
    MIT — "The shortest license that works."** 16. Star-history image.

`README.es.md` and `README.ko.md` are full translations of the same content
(keep the tables/numbers identical; translate the prose).

---

## 2. docs/

### `docs/agent-portability.md`
States the architecture: "Ponytail is an agent-portable skill distribution. The
skills in `skills/` hold the core behavior; host-specific files are adapters."
Contains the **host → files** table (reproduce every row):

| Host | Files | Tier |
|------|-------|------|
| Claude Code | `.claude-plugin/plugin.json`, `commands/`, `hooks/claude-codex-hooks.json`, `hooks/` | full plugin (activation, mode tracking, commands, statusline) |
| Codex | `.codex-plugin/plugin.json`, `hooks/claude-codex-hooks.json`, `hooks/`, `skills/` | plugin + hooks |
| OpenCode | `.opencode/plugins/ponytail.mjs`, `.opencode/command/`, `hooks/`, `skills/` | server plugin (`experimental.chat.system.transform`) |
| pi | `pi-extension/`, `skills/`, `hooks/` | package extension |
| Gemini CLI | `gemini-extension.json`, `AGENTS.md`, `commands/`, `skills/` | extension (no `hooks/hooks.json` — uses Claude/Codex event names) |
| Cursor | `.cursor/rules/ponytail.mdc` | always-on rule |
| Windsurf | `.windsurf/rules/ponytail.md` | rule |
| Cline | `.clinerules/ponytail.md` | rule |
| GitHub Copilot (editor) | `.github/copilot-instructions.md` | instructions |
| GitHub Copilot CLI | `.github/plugin/`, `AGENTS.md`, `.github/copilot-instructions.md`, `~/.copilot/copilot-instructions.md` | plugin + instruction fallback |
| Antigravity | `AGENTS.md` (or `.agents/rules/`) | instructions |
| CodeWhale | `AGENTS.md` | instructions |
| Swival | `.swival/skills/`, `AGENTS.md` | skills |
| VS Code + Codex ext. | `AGENTS.md` | instructions |
| Kiro | `.kiro/steering/ponytail.md` | steering |
| Generic | `AGENTS.md` or `skills/*/SKILL.md` | — |

Then the **Adapter Rule** ("keep adapters thin — point hosts at the existing
`skills/` and `hooks/`; for instruction-only hosts keep the copied rule text
aligned with `AGENTS.md`") and a **Portable Behavior** list naming all six
skills.

### `docs/platform-native.md`
The lazy dev's reference: "*does the platform already do this?*" Tables of native
features to reach for before a dependency. Include at least:
- **HTML form controls:** date picker→`<input type="date">`, time→`type="time"`,
  color→`type="color"`, range slider→`type="range"`, progress bar→`<progress>`,
  meter/gauge→`<meter>`. Also: details/summary for accordions, dialog for modals,
  native form validation (`required`, `pattern`, `type="email"`).
- **CSS over JS** (sticky, scroll-snap, `:has()`, grid/flex over layout libs).
- **Stdlib** sections for JS (`Intl`, `URL`, `structuredClone`, `crypto.randomUUID`)
  and Python (`itertools`, `collections`, `functools.lru_cache`, `pathlib`,
  `csv`, `datetime`). Each row: "You think you need X → what the platform has."

---

## 3. examples/ (11 before/after files + README)

Each `examples/*.md` shows the **same task answered by the same model** with no
skill vs with ponytail — verbatim benchmark output, not hand-written. Header
note: Claude Haiku 4.5, temperature 1, source `benchmarks/output.json`,
reproduce with `npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`.

`examples/README.md` carries the index table:

| Example | Without (LOC) | With (LOC) |
|---|--:|--:|
| Email Validation | 75 | 3 |
| Debounce | 116 | 10 |
| CSV Sum | 20 | 3 |
| Countdown Timer | 267 | 9 |
| Rate Limiting | 128 | 10 |

Files: `email-validation.md`, `debounce.md`, `csv-sum.md`, `react-countdown.md`,
`rate-limit.md`, plus `deep-clone.md`, `group-by.md`, `number-formatting.md`,
`url-params.md`, `modal-dialog.md`, `infinite-scroll.md`.

**Pattern for each file** (use `email-validation.md` as the template): `# Title`,
the task in quotes, the provenance note, `## Without Ponytail, N lines of code`
with the bloated multi-version answer (regex + "advanced" + third-party + a
comparison table + a recommendation), then `## With Ponytail, M lines of code`
with the minimal version and a one-line skip note. End with `**N → M lines of
code**, same model, same prompt.` Example minimal answer for email:
```python
import re

def is_valid_email(email: str) -> bool:
    return bool(re.match(r'^[^@]+@[^@]+\.[^@]+$', email))
```
> skip note: "Skipped: RFC 5322 parser, DNS MX lookup, confirmation email. Add
> when you actually need to reject `user+tag@sub.domain.co.uk`…"

`generate-examples.mjs` (in `benchmarks/`) regenerates these from
`benchmarks/output.json`.

---

## 4. benchmarks/

A promptfoo + agentic harness measuring three **arms** (no-skill baseline,
`caveman`, ponytail) across three models and five tasks, 10 runs, median
reported. Files:

- **`promptfooconfig.yaml`** — providers
  `anthropic:messages:claude-haiku-4-5-20251001`,
  `:claude-sonnet-4-6`, `:claude-opus-4-8` (each `max_tokens: 8192,
  temperature: 1`); prompts `file://arms/baseline.js`, `…/caveman.js`,
  `…/ponytail.js`; `defaultTest` asserts `file://loc.js`. Variants:
  `promptfooconfig.gemini.yaml`, `…gpt.yaml`, `…gpt-newest.yaml`.
- **`arms/`** — `baseline.js` (task only), `caveman.js` + vendored
  `caveman-SKILL.md` (MIT, from JuliusBrussee/caveman), `ponytail.js` (prepends
  the ponytail ruleset). Each exports a promptfoo prompt function.
- **`loc.js`** — deterministic LOC counter: counts lines inside fenced code
  blocks in the model output (the headline metric).
- **`prompts.json`** — the five task prompts (email validator, JS debounce, CSV
  sum, React countdown, FastAPI rate-limit).
- **`correctness.js`** / **`correctness.test.js`** — a correctness gate: each
  task has a checker that asserts the minimal output still *works* (the
  ponytail answer must stay correct, not just short). Spawns Python for email +
  CSV checks (`python3` tried before `python`; CSV needs `pandas`).
- **`behavior.js`** / **`behavior.yaml`** — a behavior/safety gate with probes
  that detect whether safety behaviors (validation, error handling) survived;
  used to score the "safe" column.
- **`robustness-audit.js`**, **`benchmark-local.py`** (Ollama path),
  **`claude-email.js`** / **`model-email.js`** (email-task helpers),
  **`generate-examples.mjs`** (writes `examples/*.md` from `output.json`).
- **`agentic/`** — the headline agentic benchmark: `run.py` (drives a headless
  Claude Code session editing tiangolo's full-stack-fastapi-template),
  `tasks.py` (12 feature tickets), `judge.py` (scores the resulting `git diff`),
  `complete.py`, and `agentic/README.md`.
- **`results/`** — dated markdown writeups (keep the filenames as historical
  record; e.g. `2026-06-18-agentic.md` is the headline result the README cites).
- **`README.md`** — reproduce instructions (the `cp ../.env.example .env`,
  `npx promptfoo@latest eval … --env-file ../.env --repeat 10` flow; the Ollama
  flow; median results tables). Requires Node ≥ 22.22.0 for promptfoo.

---

## 5. tests/ (node --test, 10 files, no deps)

Run by `node --test tests/*.test.js`. Build each to assert the contracts already
specified in files 1–3. Coverage map:

| File | Asserts |
|------|---------|
| `hooks.test.js` | `isShellSafe` allow/deny — ordinary paths pass; the rejected-input security test vectors (`"&calc"`, `$(…)`, `;rm`) are denied (these strings are *inputs the control blocks*, not commands the code runs); each hook's stdout shape per host (native/codex/copilot) by spawning the script with crafted env + stdin; flag-file write/read/clear; default-dir resolution. |
| `hooks-windows.test.js` | the `commandWindows` entries use `$env:VAR` (not `%VAR%`) and point at scripts that actually exist in `hooks/`. |
| `behavior.test.js` | each behavior probe in `benchmarks/behavior.js` returns the right verdict on known present/absent outputs (runs without an API key). |
| `correctness.test.js` | each task checker in `benchmarks/correctness.js` passes known-good and fails known-bad output. |
| `commands.test.js` | every command the pi extension registers also ships as a Claude TOML (`commands/*.toml`) and an OpenCode `.md` (`.opencode/command/`); descriptions present. |
| `copilot-plugin.test.js` | the Copilot plugin's command surface includes the debt command; wiring is minimal. |
| `gemini-extension.test.js` | `gemini-extension.json` points `contextFileName` at `AGENTS.md` and reuses `commands/` + `skills/`. |
| `openclaw-skills.test.js` | `.openclaw/skills/` matches what `build-openclaw-skills.js` would generate (not stale) and every description is one line < 160 chars. |
| `opencode-plugin.test.js` | the `.mjs` plugin's hooks behave against structural OpenCode hook shapes (system transform appends ruleset; command persists mode); `parseCommandFile` parses frontmatter. |
| `uninstall.test.js` | `uninstall.js` removes the flag + config, removes the `statusLine` entry only when it points at ponytail's script, leaves a user's own statusline untouched, ignores missing files. |

`pi-extension/test/{extension,helpers}.test.js` and
`ponytail-mcp/test/instructions.test.js` are run by their own package `test`
scripts (the root `npm test` chains `pi-extension`).

---

## 6. Final assembly checklist

```bash
# 1. ruleset + skills (file 1), 2. hooks (file 2), 3. packaging (file 3), 4. this file
node scripts/build-openclaw-skills.js          # generate .openclaw/skills from skills/
node scripts/check-rule-copies.js              # AGENTS.md ≡ every instruction-tier copy
node scripts/check-versions.js                 # 4.8.3 everywhere
git diff --exit-code .openclaw                 # generated skills committed & fresh
npm test                                        # tests/*.test.js + pi-extension
```

When all five pass, the repo is faithfully rebuilt. It should be a working,
installable plugin on every host in the portability table — and, true to its
own ladder, no larger than it needs to be.
