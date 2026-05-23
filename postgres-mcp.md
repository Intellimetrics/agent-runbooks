# Agent Runbook — PostgreSQL & Database MCP Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to connect to, query, and inspect databases (PostgreSQL, MySQL) locally or via SSH port forwarding.

---

## 🎯 What You Get
After completing this setup, the host agent gains database inspection and query capabilities through tools such as:
- **Schema Discovery**: Inspect tables, column types, primary/foreign keys, and indexes.
- **SQL Execution**: Execute SELECT queries and (if authorized) run INSERT, UPDATE, or migration statements.
- **Data Profiles**: Retrieve sample rows or count summaries.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with inspecting database state or running migrations, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Before applying database configurations, run diagnostic checks to identify existing configurations.

1. **Check for Database CLI Client**:
   - Command: `which psql || which mysql`
2. **Identify Active Database Port**:
   - Locate if port `5432` (Postgres) or `3306` (MySQL) is active:
     - Linux: `ss -ltnp '( sport = :5432 or sport = :3306 )'`
     - macOS: `lsof -nP -i :5432 -i :3306 -sTCP:LISTEN`
     - Windows: `netstat -ano | findstr "5432 3306"`

---

## ⚙️ Phase 2: Configuration Injection
Select the configuration appropriate for the host LLM client. Merges must be performed carefully without destroying existing servers.

> [!NOTE]
> The CrystalDBA database MCP server (`postgres-mcp`) is a Python package distributed via PyPI. It must be run using `uvx` or `pipx run` rather than `npx`.

### 1. Antigravity & Claude Code
- **Antigravity Global path**: `~/.gemini/antigravity-cli/mcp_config.json`
- **Claude Code Global path**: `~/.claude.json`
- **Project-scoped path**: `.mcp.json` (at project root)

JSON snippet to merge (swap connection string with the active environment details):
```json
{
  "mcpServers": {
    "postgres-db": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "postgres-mcp",
        "postgresql://postgres:postgres@localhost:5432/dev_db"
      ]
    }
  }
}
```

### 2. Codex
- **Global path**: `~/.codex/config.toml`
- **Project-scoped path**: `.codex/config.toml`

TOML table to merge:
```toml
[mcp_servers.postgres_db]
command = "uvx"
args = [
  "postgres-mcp",
  "postgresql://postgres:postgres@localhost:5432/dev_db"
]
```

---

## 🚀 Phase 3: Background Service Automation

### Scenario A — Start a Local Dev Database (Docker)
If no local database is running and Docker is available:
```bash
docker run --name pg-dev \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_DB=dev_db \
  -p 5432:5432 \
  -d postgres:16-alpine
```

### Scenario B — Port-Forward Remote Database (Local Agent, Remote DB)
If the database lives on a remote staging server, establish a local forward:
```bash
ssh -N -f -o ExitOnForwardFailure=yes -L 5432:localhost:5432 user@db-server
```

---

## 🩺 Phase 4: Health Check & Verification

1. **Probe Connection via `pg_isready` (PostgreSQL only)**:
   - Command: `pg_isready -h localhost -p 5432`
   - Expect: `localhost:5432 - accepting connections` (Note: no leading slash)
2. **Dry-Run MCP Server Connection**:
   - Command: `uvx postgres-mcp postgresql://postgres:postgres@localhost:5432/dev_db --access-mode=unrestricted`
   - Expect: Process runs and awaits JSON-RPC input (doesn't exit immediately with a connection failure).

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: The troubleshooting steps below contain container destruction (`docker rm`) and process termination (`kill`) commands. You **MUST** prompt the human operator for explicit confirmation before running these destructive actions.

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **`pg_isready` reports no response** | Database service is down or port is blocked. | 1. Check active processes: `ps aux \| grep postgres`. <br>2. Check Docker containers: `docker ps -a \| grep pg-dev`. <br>3. If container exists but stopped, restart: `docker start pg-dev`. |
| **`docker run` fails**: Name conflict. | A container named `pg-dev` already exists. | 1. **Destructive Action** (wipes container database state): `docker rm -f pg-dev`. <br>2. Re-run Phase 3 Scenario A launch command. |
| **MCP reports `Authentication failed`** | Incorrect database user or password. | Verify username and password match database environment config. In development Docker, default is `postgres`/`postgres`. |

---

## 🚫 Out of Scope
- Production database replication and clustering setups.
- Database backups and restoration management.
- Exposing databases to public network interfaces directly (non-secure).
