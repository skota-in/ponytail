# Token Analysis — your 4-tool agent setup (MEASURED)

Goal: **less tokens, smarter work.** Updated with the real measurement. The story turned out far
simpler than the inventory suggested: **one MCP server is the whole problem.**

---

## The result in one line

> **The GitLab MCP in OpenCode = ~9,000 tokens on every prompt — ~55% of your entire always-on
> budget across all four tools.** Fix that one server and you've solved the token problem.

Everything else combined — all instruction files, all 36 Copilot skill descriptions, every other
MCP — is only **~6,500–7,500 tokens/prompt.**

---

## Always-on vs on-demand (now measured)

| | Tokens / prompt | Share |
|---|---|---|
| **Always-on** (every prompt, all 4 tools) | **~15,000–16,500** | ~14% |
| On-demand (skill bodies, subagents, commands) | the rest of the ~110K | ~86% |

The audit's "110K" was worst-case. **~85–86% of it only loads when a task uses it** — so your 36
Copilot skills (~85K) cost nothing until you trigger one. Confirmed.

### Always-on, ranked (biggest first)

| # | Source | Tool | ~tokens/prompt |
|---|--------|------|---------------:|
| 1 | **GitLab MCP — 102 tool schemas (~36 KB JSON)** | OpenCode | **~9,000** |
| 2 | 36 Copilot skill descriptions | Copilot | ~2,560 |
| 3 | `CLAUDE.md` | Claude | ~833 |
| 4 | `AGENTS.md` | Codex | ~833 |
| 5 | `AGENTS.md` | OpenCode | ~833 |
| 6 | `copilot-instructions.md` | Copilot | ~651 |
| 7 | Codex system skill descs (5) | Codex | ~486 |
| 8 | Codex user skill descs (6) | Codex | ~309 |
| 9 | Claude user skill descs (6) | Claude | ~227 |
| – | `node_repl` MCP (est.) | Codex | ~300–1,500 |
| – | `dnr-check-docs` MCP — **DEAD, fails to register** | all 4 | **0** |
| – | Codex marketplace plugins (desktop UI, not CLI) | Codex | 0 |

**Per tool:** OpenCode ~9,833 · Copilot ~3,210 · Codex ~1,900–3,100 · Claude ~1,060.

---

## What this changes about the plan

Two earlier "findings" turned out to be **non-issues**, and one became the whole game:

- ❌ *"dnr-check-docs duplicated in 4 tools"* → **non-issue.** It doesn't even start
  (`statefarm-mcp-remote` not on PATH; Claude's localhost:8080 fails health-check). Costs **0
  tokens.** Remove for hygiene, not for tokens.
- ❌ *"36 Copilot skills = 85K"* → **on-demand.** Only their descriptions (~2,560) are always-on.
  Trimming all 36 descriptions saves ~2,500 *at most* — small.
- ✅ **GitLab MCP = ~9,000/prompt, 55% of the budget.** This is the one that matters.

---

## The fix — GitLab MCP (do this first)

The server exposes **102 tools**; its own npx startup log even warns about *duplicate* tools
across toolsets ("get_branch defined in merge_requests and branches") — so toolset filtering is
supported and overdue. Three routes, pick by how you work:

| Option | Saves | Trade-off |
|---|---|---|
| **Filter toolsets** to just what you use (e.g. `merge_requests` + `repositories`) | **~6,000/prompt** | keep MCP, lose tools you don't call |
| **Project-scope it** — move `gitlab` from global to a per-project `opencode.json` in GitLab repos only | **~9,000/prompt on all non-GitLab work** | must add it per GitLab repo |
| **Disable MCP**, use `gh`/`git`/`glab` CLI instead | **~9,000/prompt always** | lose MCP convenience |

**Recommended:** filter toolsets to the ones you actually use. Biggest saving for the least
disruption, and it also makes the model *smarter* — 102 tools is decision-noise that degrades tool
selection. Fewer, relevant tools = cheaper **and** better choices. (Confirm the exact filter flag
against the package's own docs — `fresh-docs` rule — since it varies by GitLab-MCP package.)

This single change drops the all-tools always-on budget from ~15–16.5K to **~9–10K/prompt.**

---

## Everything else (small, do when convenient)

- **Remove the dead `dnr-check-docs` MCP** from all four (0 tokens, pure hygiene + kills a startup
  hang). Or fix its package path if you actually want SF-docs lookup.
- **One shared instruction file.** The three identical `AGENTS.md`/`CLAUDE.md` (~833 each) → one
  canonical source. Swap the Ponytail copy for the **Lean-lite doctrine** (`BOOTSTRAP.user.lite.md`):
  ~561 vs ~833, and it adds the safety guardrails + `fresh-docs` + readability floor. ~270/tool +
  one file to maintain.
- **Trim Copilot skill descriptions** only if you've got dead skills — ~2,500 ceiling, low priority.
- **Model catalog (39 → ~10):** tidiness only; it's CLI config, *not* per-prompt tokens.

---

## ⚠️ Safety (unchanged, still worth acting on)

- **`WorktreeRemove` runs `tofu destroy -auto-approve`** in Claude — removing a worktree tears down
  Terraform infra with no confirm. Verify it's scoped to throwaway sandboxes.
- **Don't delete the `@statefarm/*` auth plugins or provider config** — that's your model
  authentication, not bloat. (See the "don't nuke everything" note: blanket-deleting `.claude`/
  `.opencode`/etc. risks locking you out of the models.)
- `GITLAB_PERSONAL_ACCESS_TOKEN` stays in env — never inline it into `opencode.json`.

---

## Bottom line

You don't need a big cleanup. You need **one config change.** Filter the GitLab MCP toolset →
~6,000 tokens/prompt back **and** sharper tool use, with everything else left intact. Then the
optional housekeeping trims another ~1–2K. Apply it with **`prompt-apply.md`** (backs up first,
confirms each change, won't touch auth).
