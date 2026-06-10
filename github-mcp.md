# Agent Runbook — GitHub MCP Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to configure and run the GitHub MCP integration to automate issue management, search, and pull requests.

---

## 🎯 What You Get
After completing this setup, you gain tools to interface directly with GitHub repositories, including:
- **Repository Management**: Retrieve file contents, inspect commits, search code, and get repository metadata.
- **Issues & PRs**: Create, update, and search GitHub issues, add comments, and create/review pull requests.
- **User Actions**: Run searches across repositories, query user details, and check pull request statuses.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with creating a pull request, searching issues, or fetching repository data, execute the phases below in sequence.

> [!TIP]
> **Check whether you need this at all.** If your agent already has shell access, the `gh` CLI covers most of this surface (issues, PRs, search, `gh api`) using the user's existing login. Set up the MCP server only when shell access is restricted or you need the typed tool surface.

---

## 🔍 Phase 1: Environment Discovery
Before applying configurations, run diagnostic checks to identify existing configurations.

1. **Verify if `gh` CLI is installed**:
   - Command: `which gh`
   - Expect: `/usr/bin/gh` or similar.
2. **Verify Authentication Status**:
   - Command: `gh auth status`
   - Validation: The output should state `Logged in to github.com` and indicate active scopes.
3. **Verify Environment Variables**:
   - Command: `echo "${GITHUB_PERSONAL_ACCESS_TOKEN:-$GITHUB_TOKEN}"`
   - Expect: A non-empty token string on stdout.
4. **Verify Docker (required for the local server form)**:
   - Command: `docker info`
   - Expect: Daemon details. If Docker is unavailable, use the hosted remote endpoint (Phase 2, Claude Code alternative) or fall back to `gh` directly.

---

## ⚙️ Phase 2: Configuration Injection
Select the configuration appropriate for the host LLM client. Merges must be performed carefully without destroying existing servers.

> [!IMPORTANT]
> **Do NOT install the legacy npm server.** `@modelcontextprotocol/server-github` is hard-deprecated on npm ("Package no longer supported"). Use GitHub's **official server** instead, in one of two forms:
> 1. **Local Docker** (works with every client below): image `ghcr.io/github/github-mcp-server`, authenticated via **`GITHUB_PERSONAL_ACCESS_TOKEN`**.
> 2. **Hosted remote endpoint** (no local install; for clients with HTTP/streamable MCP support, e.g. Claude Code): `https://api.githubcopilot.com/mcp/` with the PAT as a Bearer token.
> 
> Environment variable interpolation (e.g. `${VAR}`) is natively supported in Claude Code's `.mcp.json` but is not supported by Codex's `config.toml` or Antigravity's `mcp_config.json`.

### 1. Claude Code
- **Global path**: `~/.claude.json`
- **Project-scoped path**: `.mcp.json` (at project root)

JSON snippet to merge (local Docker form):
```json
{
  "mcpServers": {
    "github-mcp": {
      "type": "stdio",
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PERSONAL_ACCESS_TOKEN}"
      }
    }
  }
}
```

Alternative (hosted remote, no Docker required):
```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer $GITHUB_PERSONAL_ACCESS_TOKEN"
```

### 2. Antigravity
- **Global path**: `~/.gemini/antigravity-cli/mcp_config.json`
- **Project-scoped path**: `.mcp.json`

JSON snippet to merge (replace `"your_token_here"` with your actual Personal Access Token):
```json
{
  "mcpServers": {
    "github-mcp": {
      "type": "stdio",
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "your_token_here"
      }
    }
  }
}
```

### 3. Codex
- **Global path**: `~/.codex/config.toml`
- **Project-scoped path**: `.codex/config.toml`

TOML table to merge (replace `"your_token_here"` with your actual Personal Access Token):
```toml
[mcp_servers.github_mcp]
command = "docker"
args = ["run", "-i", "--rm", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server"]
# Note: Codex TOML config files do not support environment variable expansion.
# You MUST replace "your_token_here" with your actual Personal Access Token.
env = { "GITHUB_PERSONAL_ACCESS_TOKEN" = "your_token_here" }
```

---

## 🚀 Phase 3: Background Service Automation

### 1. Generate Personal Access Token (PAT)
If no token is set:
- Inform the human operator to navigate to [GitHub Tokens page](https://github.com/settings/tokens) and generate a token with permissions: `repo`, `read:org`, `user`.
- Set the token in the terminal session:
  ```bash
  export GITHUB_PERSONAL_ACCESS_TOKEN=your_token_here
  ```

### 2. Alternative: Extract Token from `gh` CLI
If the developer is logged in via `gh`, extract their active session token:
```bash
export GITHUB_PERSONAL_ACCESS_TOKEN=$(gh auth token)
```

---

## 🩺 Phase 4: Health Check & Verification

1. **Verify API Access via `gh`**:
   - Command: `gh api user --jq .login`
   - Expect: The authenticated username is printed on stdout.
2. **Verify MCP Server Invocation**:
   - Command: `docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server`
   - Expect: The image pulls (first run), the process starts, announces the stdio server on stderr, and awaits JSON-RPC input. It must not exit with a token verification error. Terminate with Ctrl-C / EOF.

---

## 🛠️ Phase 5: Automated Troubleshooting

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Token is empty or missing** | The variable was not exported, or the agent is in a sub-shell. | 1. Extract from gh CLI: `export GITHUB_PERSONAL_ACCESS_TOKEN=$(gh auth token)`. <br>2. If that fails, prompt the user to paste their token and run: `read -s token; export GITHUB_PERSONAL_ACCESS_TOKEN=$token`. |
| **HTTP 401: Bad Credentials** | The personal access token has expired or is invalid. | 1. Verify token scope and validity. <br>2. Request user to regenerate token on GitHub. |
| **HTTP 403: Rate Limit Exceeded** | The agent has executed too many GitHub API requests. | 1. Wait for reset (usually up to 60 minutes). <br>2. Check active rate limits: `curl -i -H "Authorization: token $GITHUB_PERSONAL_ACCESS_TOKEN" https://api.github.com/users/octocat`. |

---

## 🚫 Out of Scope
- GitHub Enterprise servers with custom domains (requires additional configuration parameters not detailed here).
- Managing fine-grained organization settings or billing details.
- Deleting production repositories.
