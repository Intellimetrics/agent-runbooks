# Agent Runbook — Plan-to-Plan Meta Workflow

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to break down large, ambiguous requests into smaller, actionable planning sessions.

---

## 🎯 What You Get
When asked to implement a large epic or complex feature, this runbook ensures the agent stays in "Planning Mode" to decompose the work rather than attempting to write all the code at once.
- **Task Decomposition**: Break large projects into component-level tasks.
- **Explicit Handoffs**: Identify tasks that require their own dedicated planning session.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. Use this when the user asks you to "plan the implementation of [Large Project]" or explicitly references the "plan to plan" workflow.

---

## 🔍 Phase 1: Environment Discovery

1. **Identify Project Context**:
   - Command: `cat README.md` or `cat docs/architecture.md`
   - Goal: Understand the current system architecture before proposing new components.

---

## ⚙️ Phase 2: Configuration Injection

Generate a high-level `epic.md` or `architecture-proposal.md` file in the repository (or the agent's artifacts directory). This file must contain:
1. **Executive Summary**: The goal of the large project.
2. **System Architecture**: High-level component interactions.
3. **Task Breakdown**: A bulleted list of tasks.

---

## 🚀 Phase 3: Background Service Automation

When decomposing the tasks in `epic.md`, the agent MUST evaluate the complexity of each task. 

If a task is complex (e.g., requires architectural decisions, new libraries, or touches many files), the agent MUST append the exact string:
**`[Requires Planning Session]`**

Example Task List:
- `[ ]` Update database schema for User model
- `[ ]` Implement robust authentication service `[Requires Planning Session]`
- `[ ]` Build frontend login UI

---

## 🩺 Phase 4: Health Check & Verification

1. **Present the Plan**:
   - Action: Halt execution and ask the user to review the generated `epic.md` (or artifact).
   - Expect: User approval before moving to implement *any* of the tasks.

---

## 🛠️ Phase 5: Automated Troubleshooting

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Scope Creep** | The user asks for a massive addition during the review. | 1. Acknowledge the addition. <br>2. Add it as a new task with `[Requires Planning Session]`. |
| **Agent begins writing code early** | The agent failed to stay in Planning Mode. | 1. Discard code changes (e.g., `git reset --hard`). <br>2. Refocus strictly on updating the `epic.md`. |

---

## 🚫 Out of Scope
- Actually writing the implementation code for the `[Requires Planning Session]` tasks. Those must be handled in separate conversational turns or subagent invocations.
