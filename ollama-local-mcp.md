# Agent Runbook — Ollama Local Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to install, run, and integrate a local Ollama server for private offline model inference.

---

## 🎯 What You Get
After completing this setup, you gain the ability to call local models for private developer workflows and multi-agent reviews through tools that expose:
- **Local Model Generation**: Prompt local models (e.g. `qwen2.5-coder`, `llama3.1`, `mistral`) for completion or reasoning tasks.
- **Cost Minimization**: Run low-stakes or high-volume agent reviews at zero API token cost.
- **Data Privacy**: Process files and codebases completely offline.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with deploying a local model or running private reviews, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Before applying configurations, check system capabilities and binary availability.

1. **Verify if Ollama is Installed**:
   - Command: `which ollama`
   - Expect: `/usr/local/bin/ollama` or similar.
2. **Verify GPU Availability (Optional but recommended for speed)**:
   - Command: `nvidia-smi || rocm-smi`
3. **Verify Port Availability**:
   - Check if Ollama's default port `11434` is bound:
     - Linux: `ss -ltnp 'sport = :11434'`
     - macOS: `lsof -nP -iTCP:11434 -sTCP:LISTEN`
     - Windows: `netstat -ano | findstr :11434`

---

## ⚙️ Phase 2: Configuration Injection
Select the configuration appropriate for the host LLM client. Merges must be performed carefully without destroying existing servers.

### 1. Antigravity & Claude Code
- **Antigravity Global path**: `~/.gemini/antigravity-cli/mcp_config.json`
- **Claude Code Global path**: `~/.claude.json`
- **Project-scoped path**: `.mcp.json` (at project root)

JSON snippet to merge:
```json
{
  "mcpServers": {
    "ollama-inference": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "ollama-mcp-server"
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
[mcp_servers.ollama_inference]
command = "npx"
args = [
  "-y",
  "ollama-mcp-server"
]
```

---

## 🚀 Phase 3: Background Service Automation

### 1. Start the Ollama Service
If not already running, spawn the background server daemon:
- **Linux**:
  `ollama serve >/tmp/ollama.log 2>&1 &` (or `sudo systemctl start ollama`)
- **macOS**:
  `ollama serve >/tmp/ollama.log 2>&1 &` (or open the Ollama app)

### 2. Pull the Target Model
Pull the recommended programming model:
```bash
ollama pull qwen2.5-coder:7b
```
*Note: This command runs synchronously. Wait for the model pull to complete before proceeding.*

---

## 🩺 Phase 4: Health Check & Verification

1. **Verify Daemon API Response**:
   - Command: `curl -s http://127.0.0.1:11434/api/tags`
   - Expect: A JSON response containing a `"models"` list.
2. **Verify Target Model Availability**:
   - Command: `curl -s http://127.0.0.1:11434/api/tags | grep qwen2.5-coder`
   - Expect: Non-empty match showing the model is locally cached.
3. **Run a Test Inference**:
   - Command: `curl -s -X POST http://127.0.0.1:11434/api/generate -d '{"model": "qwen2.5-coder:7b", "prompt": "respond with OK", "stream": false}'`
   - Expect: A JSON response containing a non-empty `"response"` field. (e.g. verify with `curl ... | jq -e '.response != null'`).

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: The troubleshooting steps below contain process termination (`kill`) commands. You **MUST** prompt the human operator for explicit confirmation before running these destructive actions.
> *On macOS, it is highly preferred to tell the user to quit the Ollama menu-bar application manually instead of running CLI kill commands.*

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **`curl` fails with Connection Refused** | The Ollama server is not running. | 1. Check processes: `ps aux \| grep ollama`. <br>2. Spawn the daemon: `ollama serve >/tmp/ollama.log 2>&1 &`. |
| **Timeout or slow generation** (CPU fallback). | GPU is not utilized or system has insufficient RAM. | 1. Run `nvidia-smi` to see if GPU is detected. <br>2. Ensure no other large models are loaded: `curl -s http://127.0.0.1:11434/api/ps`. <br>3. Downgrade to a smaller parameter size (e.g. `qwen2.5-coder:1.5b` or `qwen2.5-coder:0.5b`). |
| **Ollama port already bound** | A background instance is already running. | 1. Locate PID: `pgrep -f "ollama serve"`. <br>2. Terminate the process cleanly: `kill $(pgrep -f "ollama serve")` and restart service. |

---

## 🚫 Out of Scope
- Fine-tuning model weights or pre-training.
- Orchestrating multi-node model clusters.
- Exposing the Ollama daemon port to public networks without reverse proxy authentication.
