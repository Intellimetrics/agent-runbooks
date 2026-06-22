# 📂 agent-runbooks

**Portable markdown runbooks for setting up coding-agent tooling.**

Unlike traditional documentation written for humans, these runbooks are structured as **machine-executable instructions** optimized for LLM coding agents (such as Antigravity, Claude Code, and Codex). 

When you start a session, you can direct your agent to read these files to automatically configure, verify, and debug developer tools in your workspace.

---

## 🧑‍💻 For Humans: How to Use These Runbooks

Agents don't know these files exist unless you tell them. To use a runbook, start your prompt by directing the agent to read and execute the specific file.

**Example Prompts:**
- *"Agent, please read `automated-testing-runbook.md` and execute it to set up Playwright for this project."*
- *"We need to break this large feature down. Follow the workflow in `plan-to-plan-runbook.md`."*
- *"Set up the GitHub MCP server by following `github-mcp.md` so you can create PRs for me."*

Once prompted, the agent will follow the 5 phases inside the runbook without further hand-holding.

---

## 🛠️ Available Runbooks

- 🌐 **[chrome-devtools-mcp](chrome-devtools-mcp.md)** — Set up, launch, and drive a CDP-speaking browser (Chrome or Microsoft Edge) locally or over a secure SSH tunnel. Exposes standard and custom configuration blocks for Antigravity, Claude Code, and Codex.
- 🏛️ **[llm-council-mcp](llm-council-mcp.md)** — Install, set up, and run `llm-council` as an MCP server or CLI tool. Enables multi-agent consensus audits of implementation plans, specifications, task lists, and source code changes.
- 🗄️ **[postgres-mcp](postgres-mcp.md)** — Set up PostgreSQL database access via the CrystalDBA `postgres-mcp` server, local Docker databases, or SSH tunnels.
- 🦙 **[ollama-local-mcp](ollama-local-mcp.md)** — Configure local model inference (Ollama) as a private developer companion for offline generation.
- 🐙 **[github-mcp](github-mcp.md)** — Set up repository metadata access and pull request creation via GitHub's official `github-mcp-server` (local Docker or hosted remote).
- **[hermes-tweet-runbook](hermes-tweet-runbook.md)** - Install and verify Hermes Tweet as a native Hermes Agent X/Twitter toolset with read-first and approval-gated action workflows.
- 🌱 **[spec-kit-sdd](spec-kit-sdd.md)** — Scaffolds Spec-Driven Development (SDD) inside active repositories using Specify CLI.
- 🧪 **[automated-testing-runbook](automated-testing-runbook.md)** — Set up Playwright for robust browser-based End-to-End (E2E) testing.
- 🌳 **[gitflow-runbook](gitflow-runbook.md)** — Agent-executable instructions for managing feature branches and PRs using a Gitflow-inspired model and `gh` CLI.
- 📝 **[plan-to-plan-runbook](plan-to-plan-runbook.md)** — Meta-workflow guiding agents to break down large, ambiguous requests into smaller planning sessions instead of jumping straight into coding.
- 🤖 **[orchestrator-subagent-runbook](orchestrator-subagent-runbook.md)** — Strategies and execution steps for orchestrator agents to delegate discrete tasks to subagents, preserving their 200k+ token context windows.
- 🚢 **[release-checklist-runbook](release-checklist-runbook.md)** — Test-gated release flow for projects whose version string lives in multiple files (pyproject, package `__version__`, README badge, CHANGELOG): bump every sync point, tag, push, verify.
- 📜 **[agent-instructions-runbook](agent-instructions-runbook.md)** — Wire the global and project instruction files each CLI reads (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`) and merge tool snippets idempotently without clobbering existing content.

---

## 🏗️ Runbook Structure

Each runbook follows a structured phased approach to ensure coding agents can fully automate the setup:

1. **Phase 1: Environment Discovery** — Shell commands to diagnose system capabilities (Node.js, paths, ports, active processes).
2. **Phase 2: Configuration Injection** — Exact configuration snippets (JSON/TOML) for the supported LLM agents.
3. **Phase 3: Background Service Automation** — Commands to start services, launch browsers, or establish tunnels in the background.
4. **Phase 4: Health Check & Verification** — Expected CLI test inputs and validation regex patterns.
5. **Phase 5: Automated Troubleshooting** — Mapping of symptoms to automated diagnosis and correction commands.

---

## 📄 License

MIT.
