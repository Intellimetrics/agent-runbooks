# Agent Runbook — Docker Sandbox Executor Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to configure and run a secure Docker-based sandboxed command executor to isolate test execution and dependency installation.

---

## 🎯 What You Get
After completing this setup, the host agent gains sandboxed environment execution capabilities through tools that expose:
- **Isolate Code Runs**: Run test commands, build scripts, and untrusted python/node packages in an isolated container.
- **Secure File Access**: Read and write files safely within a workspace-mounted volume without affecting the host's root system.
- **Clean Toolchains**: Compile binaries or run scripts in pre-configured operating system images (e.g., Ubuntu, Alpine).

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with running scripts, installing npm/pip packages, or compiling code, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Before applying configurations, run diagnostic checks to verify Docker availability.

1. **Verify if Docker is Installed**:
   - Command: `docker --version`
   - Expect: `Docker version ...`
2. **Verify Daemon Connectivity**:
   - Command: `docker info`
   - Expect: Output details the container count, server version, and architecture.
3. **Verify User Permissions**:
   - Command: `docker ps`
   - Expect: Returns a table of running containers. If it fails with a permission denied error, the agent lacks permissions to communicate with the docker socket.

---

## ⚙️ Phase 2: Configuration Injection
Select the configuration appropriate for the host LLM client. Merges must be performed carefully without destroying existing servers.

> [!NOTE]
> The Automata Labs `code-sandbox-mcp` is a compiled binary. Once installed locally (see Phase 3), configure your agent to call the binary command directly.

### 1. Antigravity & Claude Code
- **Antigravity Global path**: `~/.gemini/antigravity-cli/mcp_config.json`
- **Claude Code Global path**: `~/.claude.json`
- **Project-scoped path**: `.mcp.json` (at project root)

JSON snippet to merge:
```json
{
  "mcpServers": {
    "docker-sandbox": {
      "type": "stdio",
      "command": "code-sandbox-mcp",
      "args": []
    }
  }
}
```

### 2. Codex
- **Global path**: `~/.codex/config.toml`
- **Project-scoped path**: `.codex/config.toml`

TOML table to merge:
```toml
[mcp_servers.docker_sandbox]
command = "code-sandbox-mcp"
args = []
```

---

## 🚀 Phase 3: Background Service Automation

### 1. Install code-sandbox-mcp Binary
Download and run the installer script on the host:
- **Linux & macOS**:
  ```bash
  curl -fsSL https://raw.githubusercontent.com/Automata-Labs-team/code-sandbox-mcp/main/install.sh | bash
  ```
- **Windows (PowerShell)**:
  ```powershell
  irm https://raw.githubusercontent.com/Automata-Labs-team/code-sandbox-mcp/main/install.ps1 | iex
  ```

### 2. Pre-pull the Base Container Image
To avoid slow startup timeouts when the MCP server first launches, pull the execution image beforehand:
```bash
docker pull python:3.11-slim
```

### 3. Check and Synchronize Permissions (Linux hosts)
By default, files created inside the container will be owned by `root`. To match the host user's permissions, locate your active UID and GID:
- Command: `id -u && id -g`
- Note: Note down these numbers. When executing inside the sandbox container, ensure the execution commands are run as the matching UID/GID using docker flags if possible, or correct them afterward on the host.

---

## 🩺 Phase 4: Health Check & Verification

1. **Verify Binary Command Location**:
   - Command: `which code-sandbox-mcp`
   - Expect: Output displays the path to the installed binary (e.g. `/usr/local/bin/code-sandbox-mcp` or `~/.local/bin/code-sandbox-mcp`).
2. **Execute a Test Command inside Sandbox**:
   - Command: `docker run --rm -v "${PWD}:/workspace" -w /workspace python:3.11-slim python -c "print('Sandbox OK')"`
   - Expect: `Sandbox OK` on stdout.

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: The troubleshooting steps below contain container destruction (`docker kill`/`rm`) and system pruning (`docker system prune`) commands. You **MUST** prompt the human operator for explicit confirmation before running these destructive actions.

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **`Permission Denied` on `/var/run/docker.sock`** | The active user is not in the `docker` group. | Prompt the user to run: `sudo usermod -aG docker $USER` and restart their terminal session. |
| **Slow startup / MCP timeouts** | The container image is still downloading or Docker disk is full. | 1. Check disk utilization: `docker system df`. <br>2. **Destructive Action** (wipes unused containers, networks, and images): `docker system prune -f`. |
| **File permissions lock after run** | Files created inside the workspace container are owned by `root`. | **Destructive Action** (requires sudo): Prompt the user to run: `sudo chown -R $(id -u):$(id -g) .` on the host to restore file ownership. |

---

## 🚫 Out of Scope
- Kubernetes cluster execution and orchestration.
- Hosted serverless sandbox containers (e.g. AWS Fargate, e2b).
- Multi-container orchestration (use Docker Compose separately).
