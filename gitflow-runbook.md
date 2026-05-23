# Agent Runbook — Gitflow Integration

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to correctly follow a Gitflow-inspired branching model combined with GitHub CLI (`gh`).

---

## 🎯 What You Get
After completing this setup, you can robustly manage feature work and releases.
- **Structured Branches**: Use `main` for production, `develop` for integration, and `feature/*` for active work.
- **Automated PRs**: Seamlessly open Pull Requests against the correct base branch using `gh`.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with creating features or submitting work, execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Before applying configurations, check existing repository state.

1. **Verify Git Installation**:
   - Command: `git --version`
   - Expect: Git version string.
2. **Verify Current Branches**:
   - Command: `git branch -a`
   - Expect: Both `main` and `develop` branches should exist.
3. **Verify `gh` CLI**:
   - Command: `gh auth status`
   - Expect: Logged in to GitHub.

---

## ⚙️ Phase 2: Configuration Injection

Ensure that the repository has a `develop` branch. If it does not exist, create it from `main`.

```bash
git checkout main
git pull origin main
git checkout -b develop
git push -u origin develop
```

---

## 🚀 Phase 3: Background Service Automation

Follow these commands when implementing a new feature:

1. **Start a Feature**:
   ```bash
   git checkout develop
   git pull origin develop
   # Replace 'feature-name' with a descriptive, hyphenated name
   git checkout -b feature/feature-name
   ```
2. **Commit Changes**:
   ```bash
   git add .
   # Follow Conventional Commits formatting
   git commit -m "feat: descriptive message"
   ```
3. **Publish and Create PR**:
   ```bash
   git push -u origin HEAD
   gh pr create --base develop --title "feat: descriptive title" --body "Detailed description of the feature."
   ```

---

## 🩺 Phase 4: Health Check & Verification

1. **Verify Clean Working Tree**:
   - Command: `git status --porcelain`
   - Expect: No output (empty string).
2. **Verify Upstream Tracking**:
   - Command: `git rev-parse --abbrev-ref --symbolic-full-name @{u}`
   - Expect: `origin/feature/feature-name`

---

## 🛠️ Phase 5: Automated Troubleshooting

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Merge Conflict on Pull** | Upstream `develop` has conflicting changes. | 1. Halt execution and explicitly ask the user for guidance on resolving the conflict. DO NOT attempt to guess resolution. |
| **Detached HEAD** | Checked out a specific commit instead of a branch. | 1. Checkout the branch again: `git checkout feature/feature-name`. |

---

## 🚫 Out of Scope
- Managing Release or Hotfix branches (handled via separate dedicated deployment pipelines).
- Resolving complex merge conflicts autonomously.
