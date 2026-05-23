# Agent Runbook — Chrome DevTools MCP Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to set up and drive a CDP-speaking browser (Chrome, Edge) locally or over an SSH tunnel.

---

## 🎯 What You Get
After completing this setup, the host agent gains the ability to interact directly with an active browser instance using Chrome DevTools Protocol (CDP) through ~45 specialized tools, including:
- **Navigation & Querying**: `navigate_page`, `evaluate_script`, `take_snapshot`
- **User Actions**: `click`, `hover`, `type_text`, `press_key`, `drag`
- **Performance & Diagnostics**: `lighthouse_audit`, `take_memory_snapshot`, `performance_analyze_insight`
- **Session Media**: `take_screenshot`

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with setting up or debugging browser automation tools, read and execute the phases below in sequence using your workspace tools.

---

## 🔍 Phase 1: Environment Discovery
Before applying configurations, run local diagnostic commands to verify the environment.

1. **Verify Node.js Version**:
   - Command: `node --version`
   - Expect: `^20.19.0 \|\| ^22.12.0 \|\| >=23` (matching the package engine specifications)
2. **Determine Chrome/Edge Binary Location**:
   - **Linux**:
     - Command: `which google-chrome || which google-chrome-stable` (Expected output: `/usr/bin/google-chrome` or similar)
     - Command: `which microsoft-edge || which microsoft-edge-stable`
   - **macOS**:
     - Typical Chrome path: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`
     - Typical Edge path: `/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge`
   - **Windows**:
     - Typical Chrome path: `C:\Program Files\Google\Chrome\Application\chrome.exe`
     - Typical Edge path: `C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe`
3. **Verify Port Availability**:
   - Check if port `9222` (default) or `9223` (alternative) is already bound:
     - Linux: `ss -ltnp 'sport = :9222'`
     - macOS: `lsof -nP -iTCP:9222 -sTCP:LISTEN`
     - Windows: `netstat -ano | findstr :9222`

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
    "chrome-devtools": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--browser-url=http://127.0.0.1:9222"]
    }
  }
}
```

### 2. Codex
Codex uses TOML configuration.
- **Global path**: `~/.codex/config.toml`
- **Project-scoped path**: `.codex/config.toml`

TOML table to merge:
```toml
[mcp_servers.chrome_devtools]
command = "npx"
args = ["-y", "chrome-devtools-mcp@latest", "--browser-url=http://127.0.0.1:9222"]
```

---

## 🚀 Phase 3: Background Service Automation

### Scenario A — Local Chrome Launch
Since Chrome 136, `--user-data-dir` is **mandatory** for remote debugging. Execute this command to spawn the browser:
```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-cdp-profile \
  --no-first-run --no-default-browser-check \
  >/tmp/chrome-cdp.log 2>&1 &
```

> [!IMPORTANT]
> **Microsoft Edge Port Coordination**: For Microsoft Edge, replace `google-chrome` with `microsoft-edge`, change the port to `9223` (if 9222 is busy), and use `--user-data-dir=/tmp/edge-cdp-profile`. 
> If you change the debugging port to `9223`, you **MUST** update the Phase 2 MCP configuration's `--browser-url` to match: `--browser-url=http://127.0.0.1:9223`.

### Scenario B — Remote Browser over SSH Tunnel
1. **Launch browser on the remote server** (add `--headless=new` if no GUI display is present):
   ```bash
   google-chrome \
     --headless=new \
     --remote-debugging-port=9222 \
     --user-data-dir=/tmp/chrome-cdp-profile \
     --no-first-run --no-default-browser-check \
     >/tmp/chrome-cdp.log 2>&1 &
   ```
2. **Establish the tunnel from the local machine**:
   ```bash
   ssh -N -f -o ExitOnForwardFailure=yes -L 9222:localhost:9222 user@remote_host
   ```
   > [!IMPORTANT]
   > Do NOT omit `-o ExitOnForwardFailure=yes`. This ensures the SSH command exits with a non-zero code if port forwarding fails, preventing silent background failures.

### Scenario C — Local Browser, Remote CLI Agent (Reverse SSH Tunnel)
This scenario applies when the developer is working on a local machine (e.g. Windows) but the LLM CLI tools (Antigravity, Claude Code, Codex) are running on a remote development server (e.g. Linux). The agent on the remote server must drive the browser running on the developer's Windows desktop.

Because you (the agent) are running on the remote Linux server, you cannot execute commands on the developer's local Windows machine. You **MUST** output these exact instructions to the human developer and wait for them to confirm execution:

1. **Launch a clean, debuggable browser instance on Windows**:
   Open Windows Terminal (PowerShell or Command Prompt) and execute one of the following commands:
   - **Using Microsoft Edge (Recommended to avoid clobbering primary Chrome session)**:
     ```powershell
     Stop-Process -Name msedge -ErrorAction SilentlyContinue
     Start-Process msedge.exe -ArgumentList "--remote-debugging-port=9222", "--user-data-dir=$env:TEMP\edge-cdp-profile", "--no-first-run", "--no-default-browser-check"
     ```
   - **Using Google Chrome**:
     ```powershell
     Stop-Process -Name chrome -ErrorAction SilentlyContinue
     Start-Process chrome.exe -ArgumentList "--remote-debugging-port=9222", "--user-data-dir=$env:TEMP\chrome-cdp-profile", "--no-first-run", "--no-default-browser-check"
     ```
2. **Open the Reverse SSH Tunnel from Windows**:
   From Windows Terminal, connect to the remote Linux server while forwarding the remote port 9222 back to the Windows local debugging port 9222:
   ```cmd
   ssh -R 9222:localhost:9222 user@remote_linux_box
   ```
   Keep this terminal session open to maintain the tunnel. The remote LLM agent will now be able to communicate with the Windows browser on port 9222 via loopback.

---

## 🩺 Phase 4: Health Check & Verification
Run these verification commands to ensure the browser and tunnel are fully responsive before attempting tool usage.

1. **Verify HTTP Discovery Endpoint**:
   - Command: `curl -s http://127.0.0.1:9222/json/version`
   - Validation criteria: The response must contain key-value pairs for `"Browser":` and `"webSocketDebuggerUrl":`.
2. **Verify Active Targets**:
   - Command: `curl -s http://127.0.0.1:9222/json`
   - Validation criteria: The response must be a valid JSON array of open browser pages/targets.
3. **Verify Tunnel/Port Bind**:
   - Linux: `ss -ltnp 'sport = :9222'`
   - macOS: `lsof -nP -iTCP:9222 -sTCP:LISTEN`

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: The troubleshooting steps below contain process termination (`kill`) and directory cleanup (`rm -rf`) commands. You **MUST** prompt the human operator for explicit confirmation before running these destructive actions.

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **`curl` returns no output** or connection refused. | Browser crashed or debugging flag was ignored due to missing `--user-data-dir`. | 1. Check `/tmp/chrome-cdp.log` for logs.<br>2. Run `ps aux \| grep chrome` to verify processes exist.<br>3. **Destructive Action** (wipes saved browser sessions/cookies): `rm -rf /tmp/chrome-cdp-profile && google-chrome --remote-debugging-port=9222 --user-data-dir=/tmp/chrome-cdp-profile --no-first-run &`. |
| **`SingletonLock` error** in log. | A previous crash left a lockfile, or another Chrome process is sharing the profile. | 1. Search for existing processes: `ps aux \| grep chrome \| grep /tmp/chrome-cdp-profile`.<br>2. If none exist, delete the lockfile: `rm -f /tmp/chrome-cdp-profile/SingletonLock`. |
| **SSH bind failure**: `Address already in use`. | A zombie SSH tunnel is already occupying port 9222. | 1. Locate the PID: `pgrep -f "ssh.*9222:localhost:9222"`.<br>2. Terminate the process cleanly: `kill $(pgrep -f "ssh.*9222:localhost:9222")`. Use `kill -9` only as a last resort. |
| **Blank / Black screenshots** from `take_screenshot`. | Headless rendering issues under Wayland or virtual display. | Add Ozone/GL fallback flags to the chrome launch command:<br>`--ozone-platform=x11 --use-gl=swiftshader` |

---

## 🚫 Out of Scope
- Firefox or other non-CDP browsers.
- Microsoft Edge-specific settings pages (e.g. `edge://settings`), which are not fully supported by `chrome-devtools-mcp`.
- Direct remote debugging configurations without SSH tunnel forwarding (security hazard).
