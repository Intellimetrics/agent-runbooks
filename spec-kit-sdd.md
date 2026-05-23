# Agent Runbook — Spec-Driven Development Scaffolding (Spec Kit)

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to install, configure, and initialize the Spec Kit (`specify`) toolkit in a codebase to enforce structured Spec-Driven Development (SDD).

---

## 🎯 What You Get
After completing this setup, your workspace is wired for SDD, providing:
- **Phase Scaffolding**: Structured directories and commands (`/speckit.specify`, `/speckit.plan`, `/speckit.tasks`, `/speckit.implement`) to move code from specs to code.
- **Agent Skill Syncing**: Injecting specific instructions and hook configurations directly into agent skill folders (e.g. `.claude/skills/`).
- **Feature Tracking**: Automated creation and lifecycle tracking of features inside the `.specify/` tracking directory.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with initializing Spec Kit, adding lifecycle hooks, or configuring SDD rules, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Before applying SDD configurations, run diagnostic checks to identify existing configurations.

1. **Verify if `specify` CLI is Installed**:
   - Command: `which specify`
   - Expect: `/usr/local/bin/specify` or similar.
2. **Verify Python & Node environments**:
   - Command: `python3 --version && node -v`
   - Expect: Python `>= 3.11` and Node `>= 20`.
3. **Verify Git Repository presence**:
   - Command: `git rev-parse --is-inside-work-tree`
   - Expect: `true` (Spec Kit operates on git-controlled repositories).

---

## ⚙️ Phase 2: Configuration Injection
Select the configuration appropriate for the host LLM client. Merges must be performed carefully without destroying existing configs.

### 1. Spec Kit Scaffolding Structure
Spec Kit does not run as a continuous stdio daemon, but rather integrates directly into agent workspaces as local skills. 
Upon initialization, Spec Kit compiles the active rules, templates, and compiler scripts directly under the `.specify/` directory.

- **Workspace Scaffolding Targets**:
  - `specs/` — Directory holding feature specs and task lists.
  - `.specify/memory/` — Stores persistent project guidelines and developer rules.
  - `.specify/scripts/` / `.specify/templates/` — Holds the spec compilation pipeline logic.
  - Client-specific skill directories (e.g., `.claude/skills/`, `.gemini/skills/`).

---

## 🚀 Phase 3: Background Service Automation

### 1. Install Specify CLI (if missing)
Choose your preferred installation method depending on system support:
- **Using `uv` (Recommended)**:
  ```bash
  uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
  ```
- **Using `pipx`**:
  ```bash
  pipx install git+https://github.com/github/spec-kit.git
  ```

### 2. Initialize Spec Kit in Project Root
Initialize Spec Kit in the current directory. You can specify the target AI client using either the `--ai` flag or the `--integration` flag (they are mutually exclusive):
```bash
specify init . --ai gemini
```
*Note: Swap `gemini` with your client agent name (e.g. `claude`, `codex`) to generate client-specific skill manifests.*

---

## 🩺 Phase 4: Health Check & Verification

1. **Verify Binary Command Execution**:
   - Command: `specify --version`
   - Expect: Prints the active Specify CLI version on stdout.
2. **Verify Scaffolding Layout**:
   - Command: `ls -la .specify/ && ls -la specs/`
   - Expect: `.specify/` contains `memory`, `scripts`, and `templates`. `specs/` exists for storing features.
3. **Run Configuration Check**:
   - Command: `specify check`
   - Expect: All integration diagnostics return success.

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: The troubleshooting steps below contain workspace directory deletion (`rm -rf`) commands. You **MUST** prompt the human operator for explicit confirmation before running these destructive actions.

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **`specify` command not found** | The CLI was installed via uv/pipx but the binary path is not on the active shell `$PATH`. | 1. Locate tool: `find ~ -name "specify"`. <br>2. Add path: Prompt the user to export the uv/pipx bin path to their `.bashrc`/`.zshrc` (e.g. `export PATH=$PATH:$HOME/.local/bin`). |
| **Initialization fails: Files exist** | A `.specify/` directory already exists. | 1. **Destructive Action** (deletes all Spec Kit files, including project memory/constitution): `rm -rf .specify`. <br>2. Re-run Phase 3 Scenario 2 initialization command. |
| **Skill compilation failure** | The specify compiler cannot find files or has conflicting extensions. | 1. Run `specify check` to locate active errors. <br>2. Re-pull spec-kit templates cleanly. |

---

## 🚫 Out of Scope
- Writing custom Spec Kit compiler plugins.
- Setting up remote continuous deployment runners.
