# Agent Runbook — llm-council Multi-Agent Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to install, configure, and execute `llm-council` for multi-agent code/plan reviews and decision audits.

---

## 🎯 What You Get
After configuring this integration, the host agent can trigger a multi-model review panel to audit workspace artifacts.
- **Parallel Deliberation**: Solicit independent verdicts (`yes` / `no` / `tradeoff`) from multiple frontier and local models simultaneously.
- **Automated Synthesis**: Summarize peer findings, identify consensus blocks, and write markdown transcripts of votes.
- **Risk Assessment**: Estimate prompt token costs and pre-flight feasibility before running expensive review passes.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with auditing a plan, spec, task list, or implementation, use this guide to invoke a parallel panel of LLMs to verify your proposed changes.

---

## 🔍 Phase 1: Installation & CLI Discovery
Before configuring the MCP server, verify the CLI tool is available or install it.

1. **Verify if already installed**:
   - Command: `which llm-council || find . -name "llm-council"`
   - If not found on path, check for local project virtualenv (adjust to your local workspace path):
     `~/development/projects/active/llm-council/.venv/bin/llm-council --version`
2. **Install globally (if missing)**:
   - If `uv` is available: `uv tool install llm-council`
   - Otherwise: `pipx install llm-council` or `pip install llm-council`
3. **Run Environment Diagnostics**:
   - Command: `llm-council doctor`
   - Expect: All checks passed. If API endpoints fail, check environment keys (e.g. `OPENROUTER_API_KEY`).

---

## ⚙️ Phase 2: Configuration Injection
Select the configuration appropriate for the host LLM client. Merges must be performed carefully without destroying existing servers.

> [!WARNING]
> Environment variable interpolation (e.g. `${VAR}`) is natively supported in Claude Code's `.mcp.json` but may not be supported by Codex's `config.toml` or Antigravity's `mcp_config.json`. If your client does not support interpolation, you must write the literal value or run the agent CLI with the environment variable set in the parent terminal.

### 1. Antigravity & Claude Code
Both tools support `.mcp.json` (project-scoped) or global JSON configurations.
- **Antigravity Global path**: `~/.gemini/antigravity-cli/mcp_config.json`
- **Claude Code Global path**: `~/.claude.json`
- **Project-scoped path**: `.mcp.json` (at project root)

JSON snippet to merge:
```json
{
  "mcpServers": {
    "llm-council": {
      "type": "stdio",
      "command": "llm-council",
      "args": ["mcp-server"],
      "env": {
        "OPENROUTER_API_KEY": "${OPENROUTER_API_KEY}"
      }
    }
  }
}
```
*Note: If developing locally, you can direct command to `<path-to-llm-council-repo>/.venv/bin/python` and args to `["-m", "llm_council.mcp_server"]` with `"PYTHONPATH"` set in `"env"`.*

### 2. Codex
Codex uses TOML configuration.
- **Global path**: `~/.codex/config.toml`
- **Project-scoped path**: `.codex/config.toml`

TOML table to merge:
```toml
[mcp_servers.llm_council]
command = "llm-council"
args = ["mcp-server"]
# Note: If Codex does not expand the "${OPENROUTER_API_KEY}" placeholder, 
# you MUST replace it with your actual literal API key (e.g. "sk-or-...") 
# or run Codex from a terminal where OPENROUTER_API_KEY is already set.
env = { "OPENROUTER_API_KEY" = "${OPENROUTER_API_KEY}" }
```

---

## 📋 Phase 3: Mode & Participant Discovery
Run the following commands to discover configured models, API connectivity, and routing modes:

1. **List Participants and Modes**:
   - Command: `llm-council list`
   - Use the command above to dynamically read active modes and peers. Do not assume hardcoded configurations, as modes are defined by the project's `.llm-council.yaml` file.
2. **Fetch Model Catalog (optional)**:
   - Command: `llm-council models openrouter --origin us --limit 10`
   - *Note: This requires the `openrouter` subcommand.*

---

## 🚀 Phase 4: Executing Reviews & Audits
Use the council to review plans or code changes before committing them.

> [!IMPORTANT]
> **Context Syntax**: The `llm-council` CLI takes multiple context files by repeating the `--context` flag, rather than passing a single list of files to `--context_files`.

### Scenario A — Auditing a Plan or Task List
To verify a new plan (`plan.md`) against requirements, run:
```bash
llm-council run \
  --mode plan \
  --context plan.md --context specs/requirements.md \
  "Analyze the proposed plan for potential side-effects, security concerns, or architectural flaws. Respond with yes, no, or tradeoff."
```

### Scenario B — Reviewing Code Changes / Diffs
To review modified files before committing:
```bash
llm-council run \
  --mode review \
  --context src/main.py --context tests/test_main.py \
  "Review the current changes for syntax errors, logical bugs, and style conformity."
```

---

## 🛠️ Phase 5: Troubleshooting & Fallbacks

| Error Symptom | Diagnostics | Resolution Step |
| :--- | :--- | :--- |
| **`OPENROUTER_API_KEY` missing error** | Run `echo $OPENROUTER_API_KEY` or check client `.env` | Ensure the key is set and exported, or pass it explicitly in the MCP environment block. |
| **Timeout during deliberation** | Large prompt size or slow model response. | 1. Estimate cost first: `llm-council estimate --mode review "Review description..."`. (Note: question is passed positionally).<br>2. Restrict context files to high-relevance files.<br>3. Set `--tier fast` or run in `quick` mode. |
| **Model quota exhausted** | Error contains `quota_exhausted` or HTTP `429`. | Set up `fallback_chain` in local `.llm-council.yaml` or run with `--tier cheap` to bypass premium models. |

---

## 🔧 Local Customization (`.llm-council.yaml`)
To customize the jury panel, write a `.llm-council.yaml` file to your project root. Example:
```yaml
participants:
  local_llama:
    type: ollama
    model: llama3.1:8b
  deepseek_chat:
    type: openai_compatible
    base_url: https://api.deepseek.com/v1
    model: deepseek-chat
    api_key_env: DEEPSEEK_API_KEY

modes:
  custom-audit:
    peers: ["claude", "local_llama", "deepseek_chat"]
```

---

## 🚫 Out of Scope
- Direct code modification or applying modifications automatically (the council is strictly advisory and read-only).
- Managing remote server environments (use `chrome-devtools-mcp` runbook for SSH tunnels).
- Arbitrary non-development reviews.
