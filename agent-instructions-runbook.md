# Agent Runbook — Instruction-File Wiring (CLAUDE.md / AGENTS.md / GEMINI.md)

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to wire up the persistent instruction files each CLI reads — global and per-project — and to merge tool snippets into them without clobbering existing content.

---

## 🎯 What You Get
- **Correct file locations and precedence** for Claude Code, Codex, Gemini CLI, and Antigravity.
- **An idempotent merge procedure**: marker-delimited sections that can be appended, updated, and verified mechanically.
- **Load verification**: confirm the agent actually ingested the instructions instead of assuming.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with "add this to my agent instructions", installing a tool's instruction snippet (e.g. llm-council's), or diagnosing why an agent ignores standing guidance, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery

1. **Identify installed CLIs**:
   - Command: `for c in claude codex gemini agy; do which $c >/dev/null 2>&1 && echo "installed: $c"; done`
2. **Identify existing instruction files**:
   ```bash
   ls -la ~/.claude/CLAUDE.md ~/.codex/AGENTS.md ~/.gemini/GEMINI.md 2>/dev/null
   ls -la CLAUDE.md AGENTS.md GEMINI.md 2>/dev/null   # project root
   ```
3. **Know the map** (global = cross-repo guidance; project = repo-specific, committed):

| CLI | Global file | Project file | Notes |
| :--- | :--- | :--- | :--- |
| Claude Code | `~/.claude/CLAUDE.md` | `./CLAUDE.md` | Supports `@path` imports; `/memory` lists what loaded. |
| Codex | `~/.codex/AGENTS.md` | `./AGENTS.md` | Nested `AGENTS.md` in subdirectories also apply. |
| Gemini CLI | `~/.gemini/GEMINI.md` | `./GEMINI.md` | Filename configurable via `contextFileName` in `~/.gemini/settings.json`. |
| Antigravity | `~/.gemini/GEMINI.md` (shared with Gemini CLI) | Follows Gemini CLI conventions | Verify project-file behavior against your installed `agy` release. |

---

## ⚙️ Phase 2: Configuration Injection
The merge rules — these are the load-bearing part.

1. **Never overwrite.** Read the target file first; back it up before editing a global file: `cp ~/.codex/AGENTS.md ~/.codex/AGENTS.md.bak`.
2. **Delimit every tool snippet with markers** so it can be found, updated, and deduplicated later:
   ```markdown
   <!-- BEGIN <tool-name> -->
   ...snippet content...
   <!-- END <tool-name> -->
   ```
3. **Idempotency check before appending**: `grep -q "BEGIN <tool-name>" <file>` — if present, replace the content BETWEEN the markers instead of appending a second copy.
4. **Placement**: repo-specific guidance goes in the project file (committed, reaches teammates); cross-repo workflow guidance goes in the global file.
5. **Keep it short.** These files are loaded whole into every session; bloat degrades the agent. Cut anything the agent could derive from the repo itself.

---

## 🚀 Phase 3: Background Service Automation
Worked example — installing llm-council's per-CLI snippets (see `llm-council-mcp.md`). `llm-council setup` generates them under `.llm-council/` (e.g. `.llm-council/instructions/claude.md`, `codex.md`, `gemini.md`).

```bash
# Codex example — adjust source snippet and target file per CLI from the Phase 1 map
target=~/.codex/AGENTS.md
grep -q "BEGIN llm-council" "$target" 2>/dev/null || {
  printf '\n<!-- BEGIN llm-council -->\n' >> "$target"
  cat .llm-council/instructions/codex.md >> "$target"
  printf '<!-- END llm-council -->\n' >> "$target"
}
```

For Claude Code, prefer the project `CLAUDE.md` (committed) so the council wiring travels with the repo.

---

## 🩺 Phase 4: Health Check & Verification

1. **Claude Code**: run `/memory` — expect both the global and project files listed as loaded.
2. **Gemini CLI**: run `/memory show` — expect the merged instruction content.
3. **Generic sentinel test** (works on any CLI): append a temporary line — `When asked "instruction check", reply INSTRUCTIONS-OK.` — start a NEW session, ask "instruction check", expect `INSTRUCTIONS-OK`. Remove the sentinel afterwards.
4. **No duplicate sections**: `grep -c "BEGIN <tool-name>" <file>` must print `1`.

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: Global instruction files belong to the human and may contain guidance you cannot see the reasons for. Back up before editing, change only your marker-delimited section, and never rewrite a global file wholesale.

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Instructions silently ignored** | Wrong filename or location (e.g. project file not at repo root), or the session started before the file existed. | 1. Re-check the Phase 1 map paths exactly. <br>2. Restart the agent session — instruction files load at startup. |
| **Snippet appears twice** | Appended without the idempotency check. | 1. Locate copies: `grep -n "BEGIN <tool-name>" <file>`. <br>2. Delete all but one marker-delimited block. |
| **Global and project guidance conflict** | Same topic addressed in both layers. | Keep the project-level version (more specific wins in practice) and delete the global copy — do not rely on precedence rules to referee. |
| **Agent misses items in a long file** | Instruction file has grown past what the model attends to reliably. | Prune aggressively; move reference detail into separate docs the agent reads on demand, linked from a one-line pointer. |

---

## 🚫 Out of Scope
- IDE-specific rules files (Cursor `.cursorrules`, Windsurf rules) — different ecosystem.
- Enterprise managed-policy instruction files.
- MCP server registration — covered by the per-tool MCP runbooks in this collection.
