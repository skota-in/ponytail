# Token Analysis — your 4-tool agent setup

Goal: **less tokens, smarter work.** This reads your inventory (Claude Code, Codex, OpenCode,
Copilot) and ranks what to fix by *real* per-prompt savings — not the scary headline number.

---

## The one insight that changes everything

The audit's "110,000+ tokens (if all loaded simultaneously)" is a **worst case that almost never
happens.** Tokens fall into two buckets, and only one is charged on every prompt:

| Bucket | What's in it | Cost |
|---|---|---|
| **Always-on** (every single prompt) | instruction/memory files · **MCP tool schemas** of connected servers · skill *descriptions* (picker metadata) · plugin tool schemas | the bill you pay constantly |
| **On-demand** (only when used) | skill *bodies* · subagent prompts · slash commands | paid only when triggered |

Your **36 Copilot skills = ~85K tokens** are almost all *bodies* → on-demand. The giant
`dt-spans` (34K) only costs you when a Dynatrace task actually loads it. So the fix is **not**
"delete 85K of skills." The fix is to shrink what's always-on and keep the heavy bodies for when
you genuinely need them.

**The real always-on driver you can't see yet: MCP tool schemas.** Every connected MCP server
injects *all* its tool definitions into context on every request. You have 3 unique servers
configured 6 times, and the audit couldn't count their tools. That's the dominant unknown —
measure it first (see `prompt-measure.md`).

---

## Your setup at a glance

| Tool | MCP | Skills | Agents | Hooks | Instructions |
|---|---|---|---|---|---|
| Claude Code 2.1.187 | 1 — `dnr-check-docs` **(conn failed)** | 6 (~34K) | – | 2 (worktree) | `~/.claude/CLAUDE.md` ~833 tok |
| Codex 0.142.0 | 2 — `dnr-check-docs`, `node_repl` | 6 (~35K) **identical to Claude** | – | – | `~/.codex/AGENTS.md` ~833 tok **identical** |
| OpenCode 1.17.8 | 2 — `gitlab`, `dnr-check-docs` | 0 (empty) | 0 | 0 | `~/.config/opencode/AGENTS.md` ~833 tok **identical** |
| Copilot (IDE only) | 1 — `dnr-check-docs` | 36 (~85K, all unique) | 2 (leanix, servicenow) | – | `~/.copilot/copilot-instructions.md` ~651 tok |

Unique assets worth keeping: GitLab / Terraform-StateFarm / P&C / PublicCloud / CI-CD skills,
Dynatrace+LeanIX+ServiceNow (Copilot), the `gitlab` and `node_repl` MCPs, the worktree hooks.
The waste is **duplication and dead config**, not the domain knowledge.

---

## Action plan — ranked by real per-prompt saving

### 🟢 Do first — always-on wins, low risk

1. **Kill the dead MCP in Claude.** `dnr-check-docs-mcp` shows `conn failed` (http://local…).
   A broken server still costs a connection attempt and may still inject stale tool schemas.
   Remove it from Claude, or point it at the working `stdio: uv run …` definition Codex/OpenCode
   use. → removes a per-prompt schema block + a startup hang.
2. **Consolidate `dnr-check-docs` from 4 tools → 1.** It's the same SF-docs server in Claude,
   Codex, OpenCode, *and* Copilot. Keep it in the **one** tool you actually query SF docs from
   (likely Codex or OpenCode). → drops its tool schemas from 3 contexts, every prompt.
3. **Measure MCP tool counts** (`prompt-measure.md`). `gitlab`, `node_repl`, `dnr-check-docs`,
   and Codex's 6 marketplace plugins (pdf/spreadsheets/…) each inject tool schemas always-on.
   If `gitlab` exposes 30 tools you're not using, that's the biggest hidden cost here.

### 🟡 Do next — de-duplicate, single source of truth

4. **Pick ONE primary between Claude and Codex for the 6 shared skills.** `copilot-api`, `ci-cd`,
   `publiccloud-docs`, `terraform-statefarm`, `p-and-c-knowledge`, `everything-gitlab` are
   byte-identical in both. Keeping both doubles maintenance and the always-on descriptions
   (~6 × ~40 tok in the redundant tool). Delete from the secondary; you can still invoke the
   primary tool when needed.
5. **One shared instruction file.** `AGENTS.md`/`CLAUDE.md` is identical 3× (Claude, Codex,
   OpenCode). Make one canonical `~/AGENTS.md` and point the others at it (Claude: `CLAUDE.md`
   = `@~/AGENTS.md`). You already run "Ponytail" in all four — swap that copy for the **Lean Kit
   lite doctrine** from this repo (`BOOTSTRAP.user.lite.md`): ~561 tok vs ~833, and it adds the
   safety guardrails + `fresh-docs` rule + readability floor Ponytail didn't have. Net: one file
   to edit, ~270 tok lighter per tool.

### 🔵 Do when convenient — housekeeping

6. **Trim Claude's model catalog.** 39 model defs in `.claude.json` (~17K). Keep the 5–10 you
   use. ⚠️ Honesty: this is CLI config, **not** injected into the prompt — it speeds the
   model-picker and shrinks the file, but barely changes per-prompt tokens. Low priority for
   token cost, fine for tidiness.
7. **Prune genuinely-unused Copilot skills.** 36 skills' *descriptions* are always-on (~1.4K
   total) and their bodies are brutal when they load (dt-spans 34K, latency 33K, query 29K…).
   Keep the domains you use (Dynatrace/LeanIX/ServiceNow); delete any `dt-*/lx-*/sn-*` you never
   trigger. Each deletion trims always-on description weight and removes a 24–34K body from ever
   loading by accident.

---

## ⚠️ Safety flags (not token-related, but found in the sweep)

- **Destructive auto-approve hook.** Claude's `WorktreeRemove` runs `tofu destroy -auto-approve`.
  Removing a git worktree will tear down Terraform-managed infra **with no confirmation.** Verify
  this is intended and scoped to throwaway sandboxes — an accidental worktree cleanup could
  destroy real resources.
- **Dead/broken MCP** (`dnr-check-docs` conn failed in Claude) — fix or remove (see #1).
- **Secret hygiene** — `GITLAB_PERSONAL_ACCESS_TOKEN` is read from env in OpenCode (good — not
  hard-coded). Keep it that way; never inline it into `opencode.json`.

---

## The "merge" plan (what consolidation looks like)

You don't merge the *domain* skills into the doctrine — they're tool-specific and valuable as-is.
You merge the **duplicated plumbing** down to one source each:

```
Instruction doctrine : 4 copies (Ponytail) → 1 shared Lean-lite (~561 tok)  [#5]
dnr-check-docs MCP   : 4 configs           → 1 tool                          [#1,#2]
Shared 6 skills      : Claude + Codex       → 1 primary tool                  [#4]
Model catalog        : 39 models            → ~10                            [#6]
Copilot skills       : 36                    → only the domains you use       [#7]
```

Always-on result: one lean doctrine + only the MCP servers you actively use + de-duplicated skill
descriptions. Heavy skill bodies stay available but cost nothing until a task needs them — which
is exactly "less tokens, smarter work."

---

## Next step

Run **`prompt-measure.md`** in Claude Code. It's read-only and counts the one number we're
missing — tools-per-MCP-server (and per Codex plugin) — so we can put a hard token figure on the
biggest always-on cost and finish the trim with real data instead of guesses.
