# 📂 agent-runbooks

**Portable markdown runbooks for setting up coding-agent tooling.**

Unlike traditional documentation written for humans, these runbooks are structured as **machine-executable instructions** optimized for LLM coding agents (such as Antigravity, Claude Code, and Codex). 

When you start a session, you can direct your agent to read these files to automatically configure, verify, and debug developer tools in your workspace.

---

## 🛠️ Available Runbooks

- 🌐 **[chrome-devtools-mcp](chrome-devtools-mcp.md)** — Set up, launch, and drive a CDP-speaking browser (Chrome or Microsoft Edge) locally or over a secure SSH tunnel. Exposes standard and custom configuration blocks for Antigravity, Claude Code, and Codex.
- 🏛️ **[llm-council-mcp](llm-council-mcp.md)** — Install, set up, and run `llm-council` as an MCP server or CLI tool. Enables multi-agent consensus audits of implementation plans, specifications, task lists, and source code changes.

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
