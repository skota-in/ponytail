# Ponytail — Runtime & Hooks (file 2 of 4)

> _Provenance & scope: build specification for the open-source, MIT-licensed
> **ponytail** project. This file reproduces the tool's own Node.js source and
> describes how it injects a coding-style ruleset into the agent's context at
> session start (the tool's documented, opt-in purpose). All code runs locally:
> no network calls, no telemetry, no executable payload beyond the tool's own
> hooks._

The behavioral engine. **This is real logic — reproduce the source verbatim.**
All hooks are CommonJS Node (no deps), fail silent, and exit 0. Five `.js`
modules + two statusline scripts + two hook-map JSONs.

Architecture: `ponytail-config.js` (pure resolver) ← `ponytail-runtime.js`
(host detection + flag I/O) and `ponytail-instructions.js` (text builder) ←
the three event hooks (`activate`, `mode-tracker`, `subagent`). OpenCode's
`.mjs` and pi's `index.js` (file 3) `require()` config + instructions too.

> ⚠️ **Note on the BOM strip:** the BOM-stripping regexes below are written as the
> 6-character escape `/^\uFEFF/`. Emit those six literal characters
> (backslash-u-F-E-F-F) — do **not** let a paste collapse them into an actual
> invisible BOM character inside the regex. Both behave the same at runtime, but
> only the escape form is byte-identical to the source.

---

## 1. `hooks/ponytail-config.js` — shared resolver

Resolution for default mode: `PONYTAIL_DEFAULT_MODE` env → config-file
`defaultMode` → `'full'`. `VALID_MODES = ['off','lite','full','ultra','review']`,
`RUNTIME_MODES = ['off','lite','full','ultra']`.

```js
#!/usr/bin/env node
// ponytail — shared configuration resolver
//
// Resolution order for default mode:
//   1. PONYTAIL_DEFAULT_MODE environment variable
//   2. Config file defaultMode field:
//      - $XDG_CONFIG_HOME/ponytail/config.json (any platform, if set)
//      - ~/.config/ponytail/config.json (macOS / Linux fallback)
//      - %APPDATA%\ponytail\config.json (Windows fallback)
//   3. 'full'

const fs = require('fs');
const path = require('path');
const os = require('os');

const DEFAULT_MODE = 'full';
const VALID_MODES = ['off', 'lite', 'full', 'ultra', 'review'];
const RUNTIME_MODES = ['off', 'lite', 'full', 'ultra'];

function normalizeMode(mode) {
  if (typeof mode !== 'string') return null;
  const normalized = mode.trim().toLowerCase();
  return RUNTIME_MODES.includes(normalized) ? normalized : null;
}

function normalizeConfigMode(mode) {
  if (typeof mode !== 'string') return null;
  const normalized = mode.trim().toLowerCase();
  return VALID_MODES.includes(normalized) ? normalized : null;
}

function normalizePersistedMode(mode) {
  return normalizeMode(mode) || normalizeConfigMode(mode);
}

// "stop ponytail" / "normal mode" turn ponytail off, but only as a standalone
// command. Matching the phrase anywhere in the message turned it off mid-task
// for ordinary requests like "add a normal mode toggle" — so require the whole
// message to be the command, ignoring case and trailing punctuation.
function isDeactivationCommand(text) {
  const t = String(text || '').trim().toLowerCase().replace(/[.!?\s]+$/, '');
  return t === 'stop ponytail' || t === 'normal mode';
}

// ponytail: only embed the plugin install path in a statusline shell command when
// it's made of ordinary path characters. An allowlist beats escaping every shell's
// metacharacters; a hostile clone path (quotes, &, $, backtick, ;, etc.) falls back
// to manual setup instead. Allows : \ / for normal Windows and POSIX paths. Full
// per-shell escaper only if a real need appears.
function isShellSafe(p) {
  return typeof p === 'string' && /^[A-Za-z0-9 _.\-:/\\~]+$/.test(p);
}

function getConfigDir() {
  if (process.env.XDG_CONFIG_HOME) {
    return path.join(process.env.XDG_CONFIG_HOME, 'ponytail');
  }
  if (process.platform === 'win32') {
    return path.join(
      process.env.APPDATA || path.join(os.homedir(), 'AppData', 'Roaming'),
      'ponytail'
    );
  }
  return path.join(os.homedir(), '.config', 'ponytail');
}

function getConfigPath() {
  return path.join(getConfigDir(), 'config.json');
}

function getClaudeDir() {
  // ponytail: CLAUDE_CONFIG_DIR overrides ~/.claude, matching Claude Code.
  return process.env.CLAUDE_CONFIG_DIR || path.join(os.homedir(), '.claude');
}

function getDefaultMode() {
  // 1. Environment variable (highest priority)
  const envMode = process.env.PONYTAIL_DEFAULT_MODE;
  if (envMode && VALID_MODES.includes(envMode.toLowerCase())) {
    return envMode.toLowerCase();
  }

  // 2. Config file
  try {
    const configPath = getConfigPath();
    const config = JSON.parse(fs.readFileSync(configPath, 'utf8'));
    if (config.defaultMode && VALID_MODES.includes(config.defaultMode.toLowerCase())) {
      return config.defaultMode.toLowerCase();
    }
  } catch (e) {
    // Config file doesn't exist or is invalid — fall through
  }

  // 3. Default
  return DEFAULT_MODE;
}

function writeDefaultMode(mode) {
  const normalized = normalizeConfigMode(mode);
  if (!normalized) return null;

  const configPath = getConfigPath();
  fs.mkdirSync(path.dirname(configPath), { recursive: true });
  fs.writeFileSync(configPath, JSON.stringify({ defaultMode: normalized }, null, 2), 'utf8');
  return normalized;
}

module.exports = {
  DEFAULT_MODE,
  VALID_MODES,
  RUNTIME_MODES,
  getDefaultMode,
  getConfigDir,
  getConfigPath,
  getClaudeDir,
  isShellSafe,
  normalizeMode,
  normalizeConfigMode,
  normalizePersistedMode,
  isDeactivationCommand,
  writeDefaultMode,
};
```

---

## 2. `hooks/ponytail-runtime.js` — host detection, flag I/O, hook output

Three hosts share one codebase, distinguished by env vars: **Copilot** (when
`COPILOT_PLUGIN_DATA` set), **Codex** (when `PLUGIN_DATA` set and not Copilot),
**native Claude** (neither). The flag file `.ponytail-active` lives in the
host's state dir. Each host wants a different stdout shape from a hook —
`writeHookOutput` encodes that.

```js
const fs = require('fs');
const path = require('path');
const { getClaudeDir } = require('./ponytail-config');

const STATE_FILE = '.ponytail-active';
const isCopilot = Boolean(process.env.COPILOT_PLUGIN_DATA);
const isCodex = !isCopilot && Boolean(process.env.PLUGIN_DATA);

let stateDir = getClaudeDir();
if (isCodex) stateDir = process.env.PLUGIN_DATA;
if (isCopilot) stateDir = process.env.COPILOT_PLUGIN_DATA;

const statePath = path.join(stateDir, STATE_FILE);

function setMode(mode) {
  fs.mkdirSync(path.dirname(statePath), { recursive: true });
  fs.writeFileSync(statePath, mode);
}

function clearMode() {
  try { fs.unlinkSync(statePath); } catch (e) {}
}

// Live mode written by activate/mode-tracker. Absent flag = ponytail off.
function readMode() {
  try {
    return fs.readFileSync(statePath, 'utf8').trim() || null;
  } catch (e) {
    return null;
  }
}

function writeHookOutput(event, mode, context = '') {
  if (isCopilot) {
    // Copilot reads additionalContext on SessionStart; ignores output elsewhere.
    process.stdout.write(JSON.stringify(
      event === 'SessionStart' && context ? { additionalContext: context } : {}));
    return;
  }
  if (isCodex) {
    const output = { systemMessage: `PONYTAIL:${mode.toUpperCase()}` };
    if (context) {
      output.hookSpecificOutput = {
        hookEventName: event,
        additionalContext: context,
      };
    }
    process.stdout.write(JSON.stringify(output));
    return;
  }
  // Native Claude: SessionStart accepts raw stdout, but SubagentStart needs the
  // hookSpecificOutput JSON form or the context is dropped.
  if (event === 'SubagentStart') {
    process.stdout.write(JSON.stringify(
      { hookSpecificOutput: { hookEventName: event, additionalContext: context } }));
    return;
  }
  process.stdout.write(context);
}

module.exports = {
  clearMode,
  isCodex,
  isCopilot,
  readMode,
  setMode,
  writeHookOutput,
};
```

---

## 3. `hooks/ponytail-instructions.js` — build ruleset text, filtered by mode

Reads `skills/ponytail/SKILL.md`, strips frontmatter, and drops the
intensity-table rows and worked-example lines whose mode label ≠ the active
mode (bullets whose label is a *rule*, not a mode, are kept verbatim). `review`
is an "independent mode" — returns a one-line pointer to the review skill. If
the SKILL.md can't be read, falls back to a hard-coded compact ruleset.

```js
#!/usr/bin/env node
// Shared Ponytail instruction builder for Claude hooks and Pi extension.

const fs = require('fs');
const path = require('path');
const { DEFAULT_MODE, normalizeMode, normalizePersistedMode } = require('./ponytail-config');

const INDEPENDENT_MODES = new Set(['review']);
const SKILL_PATH = path.join(__dirname, '..', 'skills', 'ponytail', 'SKILL.md');

function filterSkillBodyForMode(body, mode) {
  const effectiveMode = normalizeMode(mode) || DEFAULT_MODE;
  const withoutFrontmatter = String(body || '').replace(/^---[\s\S]*?---\s*/, '');

  // Only the intensity table rows and worked examples are mode-specific, and
  // both are keyed by a mode name (lite/full/ultra). A bullet whose label is
  // not a mode — e.g. "No unrequested abstractions: ..." — is a normal rule
  // and must be kept verbatim.
  return withoutFrontmatter
    .split(/\r?\n/)
    .filter((line) => {
      const tableLabel = line.match(/^\|\s*\*\*(.+?)\*\*\s*\|/);
      if (tableLabel) {
        const labelMode = normalizeMode(tableLabel[1].trim());
        if (labelMode) return labelMode === effectiveMode;
      }

      const exampleLabel = line.match(/^-\s*([^:]+):\s*/);
      if (exampleLabel) {
        const labelMode = normalizeMode(exampleLabel[1].trim());
        if (labelMode) return labelMode === effectiveMode;
      }

      return true;
    })
    .join('\n');
}

function getFallbackInstructions(mode) {
  return 'PONYTAIL MODE ACTIVE — level: ' + mode + '\n\n' +
    'You are a lazy senior developer. Lazy means efficient, not careless. The best code is the code never written.\n\n' +
    '## Persistence\n\n' +
    'ACTIVE EVERY RESPONSE. No drift back to over-building. Still active if unsure. Off only: "stop ponytail" / "normal mode".\n\n' +
    'Current level: **' + mode + '**. Switch: `/ponytail lite|full|ultra`.\n\n' +
    '## The ladder\n\n' +
    'Before any code, stop at the first rung that holds (the ladder runs after you understand the problem, not instead of it — read the code it touches and trace the real flow first):\n' +
    '1. Does this need to be built at all? (YAGNI)\n' +
    '2. Does it already exist in this codebase? Reuse what is already here, do not re-write it.\n' +
    '3. Does the standard library do this? Use it.\n' +
    '4. Does a native platform feature cover it? Use it.\n' +
    '5. Does an already-installed dependency solve it? Use it.\n' +
    '6. Can this be one line? Make it one line.\n' +
    '7. Only then: write the minimum code that works.\n\n' +
    'Bug fix = root cause, not symptom: grep every caller of the function you touch and fix the shared function once (a smaller diff than one guard per caller); patching only the path the ticket names leaves a sibling caller broken.\n\n' +
    '## Rules\n\n' +
    'No abstractions that were not requested. No avoidable dependencies. No boilerplate nobody asked for. ' +
    'Deletion over addition. Boring over clever. Fewest files possible. ' +
    'Ship the lazy version and question the complex request in the same response — never stall. ' +
    'Between two same-size stdlib options, pick the one correct on edge cases. ' +
    'Mark intentional simplifications with a `ponytail:` comment — a shortcut with a known ceiling names the ceiling and the upgrade path in the comment.\n\n' +
    '## Output\n\n' +
    'Code first. Then at most three short lines: what was skipped, when to add it. ' +
    'If the explanation is longer than the code, delete the explanation. ' +
    'Explanation the user explicitly asked for is not debt, give it in full.\n\n' +
    '## When NOT to be lazy\n\n' +
    'Never simplify away: understanding the problem (read it fully and trace the real flow before picking a rung — a small diff you do not understand is just laziness dressed up as efficiency), input validation at trust boundaries, error handling that prevents data loss, ' +
    'security measures, accessibility basics, the calibration real hardware needs (the platform is never the spec ideal), anything the user explicitly asked to keep. ' +
    'Lazy code without its check is unfinished: non-trivial logic leaves ONE runnable check behind (assert-based demo/self-check or one small test file; no frameworks). Trivial one-liners need no test.\n\n' +
    '## Boundaries\n\n' +
    'Ponytail governs what you build, not how you talk. "stop ponytail" or "normal mode": revert. Level persists until changed or session end.';
}

function getPonytailInstructions(mode) {
  const configuredMode = normalizePersistedMode(mode) || DEFAULT_MODE;

  if (INDEPENDENT_MODES.has(configuredMode)) {
    return 'PONYTAIL MODE ACTIVE — level: ' + configuredMode + '. Behavior defined by /ponytail-' + configuredMode + ' skill.';
  }

  const effectiveMode = normalizeMode(configuredMode) || DEFAULT_MODE;

  try {
    return 'PONYTAIL MODE ACTIVE — level: ' + effectiveMode + '\n\n' +
      filterSkillBodyForMode(fs.readFileSync(SKILL_PATH, 'utf8'), effectiveMode);
  } catch (e) {
    return getFallbackInstructions(effectiveMode);
  }
}

module.exports = {
  filterSkillBodyForMode,
  getFallbackInstructions,
  getPonytailInstructions,
};
```

---

## 4. `hooks/ponytail-activate.js` — SessionStart

On every session start: (1) resolve mode; if `off`, clear flag and emit `OK`
(native) / `''` (codex/copilot) and exit. (2) Otherwise write the flag, (3) emit
the mode-filtered ruleset as hidden SessionStart context, and (4) on native
Claude only, if no `statusLine` is configured in `~/.claude/settings.json`,
append a setup nudge (using a shell-safe command snippet, or manual-setup text
if the install path has shell metacharacters). All steps fail silent.

```js
#!/usr/bin/env node
// ponytail — Claude Code SessionStart activation hook
//
// Runs on every session start:
//   1. Writes flag file at $CLAUDE_CONFIG_DIR/.ponytail-active (defaults to ~/.claude; statusline reads this)
//   2. Emits ponytail ruleset as hidden SessionStart context
//   3. Detects missing statusline config and emits setup nudge

const fs = require('fs');
const path = require('path');
const { getDefaultMode, getClaudeDir, isShellSafe } = require('./ponytail-config');
const { getPonytailInstructions } = require('./ponytail-instructions');
const {
  clearMode,
  isCodex,
  isCopilot,
  setMode,
  writeHookOutput,
} = require('./ponytail-runtime');

const claudeDir = getClaudeDir();
const settingsPath = path.join(claudeDir, 'settings.json');

const mode = getDefaultMode();

// "off" mode — skip activation entirely, don't write flag or emit rules
if (mode === 'off') {
  clearMode();
  const hookOutput = (isCodex || isCopilot) ? '' : 'OK';
  writeHookOutput('SessionStart', 'off', hookOutput);
  process.exit(0);
}

// 1. Write flag file
try {
  setMode(mode);
} catch (e) {
  // Silent fail -- flag is best-effort, don't block the hook
}

// 2. Emit the ponytail ruleset, filtered to the active intensity level.
let output = getPonytailInstructions(mode);

// 3. Detect missing statusline config — nudge Claude to help set it up
if (!isCodex && !isCopilot) try {
  let hasStatusline = false;
  if (fs.existsSync(settingsPath)) {
    // Strip UTF-8 BOM some editors prepend on Windows (breaks JSON.parse)
    const raw = fs.readFileSync(settingsPath, 'utf8').replace(/^\uFEFF/, '');
    const settings = JSON.parse(raw);
    if (settings.statusLine) {
      hasStatusline = true;
    }
  }

  if (!hasStatusline) {
    const isWindows = process.platform === 'win32';
    const scriptName = isWindows ? 'ponytail-statusline.ps1' : 'ponytail-statusline.sh';
    const scriptPath = path.join(__dirname, scriptName);
    if (isShellSafe(scriptPath)) {
      const command = isWindows
        ? `powershell -ExecutionPolicy Bypass -File "${scriptPath}"`
        : `bash "${scriptPath}"`;
      const statusLineSnippet =
        '"statusLine": { "type": "command", "command": ' + JSON.stringify(command) + ' }';
      output += "\n\n" +
        "STATUSLINE SETUP NEEDED: The ponytail plugin includes a statusline badge showing active mode " +
        "(e.g. [PONYTAIL], [PONYTAIL:ULTRA]). It is not configured yet. " +
        "To enable, add this to ~/.claude/settings.json: " +
        statusLineSnippet + " " +
        "Proactively offer to set this up for the user on first interaction.";
    } else {
      // ponytail: install path has shell metacharacters — don't embed it in a
      // command snippet; have the agent wire it up by hand instead.
      output += "\n\n" +
        "STATUSLINE SETUP NEEDED: The ponytail plugin includes a statusline badge showing active mode. " +
        "Its install path contains characters unsafe to embed in a shell command, so configure it manually: " +
        "add a statusLine command of type \"command\" that runs " + scriptName +
        " from the plugin's hooks directory to ~/.claude/settings.json, quoting/escaping the path for your shell. " +
        "Proactively offer to set this up for the user on first interaction.";
    }
  }
} catch (e) {
  // Silent fail — don't block session start over statusline detection
}

try {
  writeHookOutput('SessionStart', mode, output);
} catch (e) {
  // Silent fail — stdout closed/EPIPE at hook exit must not surface as a hook failure
}
```

---

## 5. `hooks/ponytail-mode-tracker.js` — UserPromptSubmit

Reads the submitted prompt (JSON on stdin, BOM-stripped). If it starts with
`/`, `@`, or `$` + `ponytail`, parse the command + arg and switch mode
(`/ponytail-review` → `review`; `/ponytail lite|full|ultra|off` → that; bare
`/ponytail` → default). Also handles namespaced `/ponytail:ponytail…` forms.
Standalone "stop ponytail"/"normal mode" clears the flag.

```js
#!/usr/bin/env node
// ponytail — UserPromptSubmit hook to track which ponytail mode is active
// Inspects user input for /ponytail commands and writes mode to flag file

const { getDefaultMode, isDeactivationCommand } = require('./ponytail-config');
const { clearMode, setMode, writeHookOutput } = require('./ponytail-runtime');

let input = '';
process.stdin.on('data', chunk => { input += chunk; });
process.stdin.on('end', () => {
  try {
    // Strip UTF-8 BOM some shells prepend when piping (breaks JSON.parse)
    const data = JSON.parse(input.replace(/^\uFEFF/, ''));
    const prompt = (data.prompt || '').trim().toLowerCase();

    // Match /ponytail commands
    if (/^[/@$]ponytail/.test(prompt)) {
      const parts = prompt.split(/\s+/);
      const cmd = parts[0].replace(/^[@$]/, '/');
      const arg = parts[1] || '';

      let mode = null;

      if (cmd === '/ponytail-review' || cmd === '/ponytail:ponytail-review') {
        mode = 'review';
      } else if (cmd === '/ponytail' || cmd === '/ponytail:ponytail') {
        if (arg === 'lite') mode = 'lite';
        else if (arg === 'full') mode = 'full';
        else if (arg === 'ultra') mode = 'ultra';
        else if (arg === 'off') mode = 'off';
        else mode = getDefaultMode();
      }

      if (mode && mode !== 'off') {
        setMode(mode);
        writeHookOutput(
          'UserPromptSubmit',
          mode,
          'PONYTAIL MODE CHANGED — level: ' + mode,
        );
      } else if (mode === 'off') {
        clearMode();
        writeHookOutput('UserPromptSubmit', 'off', 'PONYTAIL MODE OFF');
      }
    }

    // Detect deactivation
    if (isDeactivationCommand(prompt)) {
      clearMode();
      writeHookOutput('UserPromptSubmit', 'off', 'PONYTAIL MODE OFF');
    }
  } catch (e) {
    // Silent fail
  }
});
```

---

## 6. `hooks/ponytail-subagent.js` — SubagentStart

SessionStart context never reaches subagents, so Task-spawned agents would run
ponytail-unaware. When the flag says a mode is active, inject the same ruleset
into each subagent.

```js
#!/usr/bin/env node
// ponytail — Claude Code SubagentStart hook
//
// SessionStart context is parent-thread only and never reaches subagents, so
// without this every Task-spawned agent runs ponytail-unaware (issue #252).
// When ponytail mode is active, inject the same ruleset into each subagent.

const { getPonytailInstructions } = require('./ponytail-instructions');
const { readMode, writeHookOutput } = require('./ponytail-runtime');

const mode = readMode();

// Absent flag or off → ponytail isn't active; inject nothing.
if (!mode || mode === 'off') {
  process.exit(0);
}

try {
  writeHookOutput('SubagentStart', mode, getPonytailInstructions(mode));
} catch (e) {
  // Silent fail — a stdout error at hook exit must not surface as a hook failure.
}
```

---

## 7. Statusline scripts

### `hooks/ponytail-statusline.sh`
Reads the flag at `${CLAUDE_CONFIG_DIR:-$HOME/.claude}/.ponytail-active`. Prints
nothing if absent. `[PONYTAIL]` for `full`/empty, else `[PONYTAIL:<MODE>]`
uppercased, in a muted green (ANSI 256-color 108).

```bash
#!/usr/bin/env bash
# CLAUDE_CONFIG_DIR overrides ~/.claude, matching where the hooks write the flag (issue #34)
flag="${CLAUDE_CONFIG_DIR:-$HOME/.claude}/.ponytail-active"
[ -f "$flag" ] || exit 0

mode=$(head -n1 "$flag" | tr -d '[:space:]')

if [ -z "$mode" ] || [ "$mode" = "full" ]; then
    printf '\033[38;5;108m[PONYTAIL]\033[0m'
else
    printf '\033[38;5;108m[PONYTAIL:%s]\033[0m' "$(printf '%s' "$mode" | tr '[:lower:]' '[:upper:]')"
fi
```

### `hooks/ponytail-statusline.ps1`
PowerShell equivalent: same flag path (use `$env:CLAUDE_CONFIG_DIR` else
`$HOME\.claude`), exit silently if missing, print `[PONYTAIL]` or
`[PONYTAIL:<MODE>]` uppercased. (Color optional; the badge text is what matters.)

---

## 8. Hook maps

### `hooks/claude-codex-hooks.json` (Claude Code + Codex)
Three events. Each runs the matching `.js` via `node`, has a `commandWindows`
PowerShell variant guarded by `Get-Command node`, `timeout: 5`, a
`statusMessage`, and `; exit 0` so a hook failure never blocks. `${CLAUDE_PLUGIN_ROOT}`
is the plugin root.

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/ponytail-activate.js\"; exit 0",
            "commandWindows": "if (Get-Command node -ErrorAction SilentlyContinue) { node \"$env:CLAUDE_PLUGIN_ROOT\\hooks\\ponytail-activate.js\" }",
            "timeout": 5,
            "statusMessage": "Loading ponytail mode..."
          }
        ]
      }
    ],
    "SubagentStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/ponytail-subagent.js\"; exit 0",
            "commandWindows": "if (Get-Command node -ErrorAction SilentlyContinue) { node \"$env:CLAUDE_PLUGIN_ROOT\\hooks\\ponytail-subagent.js\" }",
            "timeout": 5,
            "statusMessage": "Loading ponytail mode..."
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node \"${CLAUDE_PLUGIN_ROOT}/hooks/ponytail-mode-tracker.js\"; exit 0",
            "commandWindows": "if (Get-Command node -ErrorAction SilentlyContinue) { node \"$env:CLAUDE_PLUGIN_ROOT\\hooks\\ponytail-mode-tracker.js\" }",
            "timeout": 5,
            "statusMessage": "Tracking ponytail mode..."
          }
        ]
      }
    ]
  }
}
```

### `hooks/copilot-hooks.json` (Copilot CLI)
Copilot's schema: `version: 1`, lowercase event names, `bash`+`powershell`
keys, `timeoutSec`, `${PLUGIN_ROOT}`. Only two events (no subagent hook).

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "type": "command",
        "bash": "node \"${PLUGIN_ROOT}/hooks/ponytail-activate.js\"",
        "powershell": "node \"${PLUGIN_ROOT}\\hooks\\ponytail-activate.js\"",
        "timeoutSec": 5
      }
    ],
    "userPromptSubmitted": [
      {
        "type": "command",
        "bash": "node \"${PLUGIN_ROOT}/hooks/ponytail-mode-tracker.js\"",
        "powershell": "node \"${PLUGIN_ROOT}\\hooks\\ponytail-mode-tracker.js\"",
        "timeoutSec": 5
      }
    ]
  }
}
```

> Continue to `03-PACKAGING-AND-SUPPORT.md`.
