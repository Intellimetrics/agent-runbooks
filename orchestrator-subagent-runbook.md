# Agent Runbook — Subagent Delegation (Context Management)

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to delegate tasks to subagents, preserving the primary orchestrator's context window.

---

## 🎯 What You Get
When executing large or prolonged projects, the main orchestrator agent risks bloating its context window (especially past 200k tokens), leading to degraded reasoning, forgotten instructions, and "sloppy" code generation.
- **Context Preservation**: Offload isolated development tasks to fresh subagents.
- **Parallel Execution**: Run multiple discrete development or research tasks simultaneously.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. Use this when the user asks you to "use subagents for development" or when you recognize that a task is a discrete unit of work that can be safely offloaded to avoid context bloat.

---

## 🔍 Phase 1: Environment Discovery

1. **Identify Subagent Capabilities**:
   - Action: Review your available tool schemas (e.g., `invoke_subagent`, `define_subagent`, or equivalent platform native capabilities).
   - Expect: Confirmation that you have the capability to spawn background agents.

---

## ⚙️ Phase 2: Configuration Injection

When delegating, you must provide the subagent with a perfectly scoped prompt. Do not assume the subagent shares your conversation history.

1. **Draft the Subagent Prompt**:
   Create a prompt containing:
   - The exact goal.
   - The specific absolute file paths to modify.
   - Strict instructions to return a summary and halt upon completion.
   
Example Prompt Structure:
```text
Task: Implement the login form UI in `src/components/Login.tsx`.
Context: Use Tailwind CSS. Do not modify the authentication backend logic.
Deliverable: Update the file and reply with a summary of the UI components added.
```

---

## 🚀 Phase 3: Background Service Automation

1. **Invoke the Subagent**:
   - Action: Use your `invoke_subagent` (or equivalent) tool to launch the agent.
   - Parameters: Assign a specific role (e.g., "Frontend Developer", "Codebase Researcher") and pass the drafted prompt.
   - Workspace: Inherit or share the parent workspace unless isolated branching is required.

2. **Yield Execution**:
   - Action: Once the subagent is launched, immediately halt your own tool execution to go idle. The system will wake you up when the subagent responds. Do NOT poll in a loop.

---

## 🩺 Phase 4: Health Check & Verification

1. **Verify Subagent Completion**:
   - Action: Upon receiving a message from the subagent, read the summary and verify the requested file changes using `view_file`, `git diff`, or running relevant tests.
   - Expect: The requested changes are accurately implemented without regressions.

---

## 🛠️ Phase 5: Automated Troubleshooting

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Subagent Hallucinates Files** | The subagent lacked exact file paths in its prompt. | 1. Send a message to the subagent with the absolute paths of the target files and instruct it to retry. |
| **Subagent Times Out / Hangs** | The subagent got stuck in a loop or encountered an interactive prompt. | 1. Use your `manage_task` or subagent management tools to kill the hung subagent. <br>2. Spawn a new subagent with explicit instructions to avoid interactive commands. |
| **Orchestrator Context Too Large** | You waited too long to delegate. | 1. Summarize your current state into a scratchpad artifact. <br>2. Offload the remaining steps immediately to a subagent to continue with a fresh context. |

---

## 🚫 Out of Scope
- Orchestrating complex hierarchical swarms where subagents spawn their own sub-subagents indefinitely (keep it flat: Orchestrator -> Subagents).
