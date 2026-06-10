# Agent Runbook — Release Checklist (Version-Synced Projects)

This document contains executable instructions for agentic coding tools (such as Antigravity, Claude Code, and Codex) to cut a release in projects whose version string lives in multiple files, without leaving any of them behind.

Motivating incident: a sibling project tagged `v0.14.1` with only `pyproject.toml` bumped — the package `__version__`, README badge, and CHANGELOG all lagged, and the repo's version-sync test failed on `main` until back-filled post-tag. Every step below exists to prevent a repeat.

---

## 🎯 What You Get
- **One pass, every sync point**: locate and bump all version declarations together, never just the one you remembered.
- **Test-gated tagging**: the tag is only created on a green full suite.
- **Post-release verification**: mechanical checks that the tag, versions, and remote all agree.

---

## 🎯 Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured specifically for LLM-based CLI agents. When tasked with "cut a release", "bump the version", or "tag vX.Y.Z", execute the phases below in sequence.

---

## 🔍 Phase 1: Environment Discovery
Enumerate the version sync points BEFORE touching anything — projects differ.

1. **Locate every declared version** (Python projects; adapt for Node — `package.json` / lockfile):
   ```bash
   grep -n '^version' pyproject.toml
   grep -rn '__version__' --include='__init__.py' .
   grep -n 'badge/version' README.md
   head -5 CHANGELOG.md
   ```
   Record each file:line. Any file showing a DIFFERENT version than the others is a pre-existing desync — surface it to the human before proceeding.
2. **Identify what is being released**:
   ```bash
   git describe --tags --abbrev=0          # last release tag
   git log $(git describe --tags --abbrev=0)..HEAD --oneline
   ```
3. **Check for a version-sync test** (run it later as the cheapest verification):
   ```bash
   grep -rln 'pyproject' tests/ | head -3
   ```
4. **Verify a clean working tree on the release branch**: `git status --porcelain` must be empty.

---

## ⚙️ Phase 2: Configuration Injection
Decide and apply the bump.

1. **Pick the semver increment** from the unreleased commits: breaking change → major, `feat:` → minor, only `fix:`/`docs:`/`chore:` → patch. If ambiguous, ask the human — do not guess major bumps.
2. **Edit EVERY location found in Phase 1** to the same new version string. Typical Python set: `pyproject.toml`, `<package>/__init__.py` `__version__`, README version badge.
3. **Write the CHANGELOG entry**: `## X.Y.Z - YYYY-MM-DD` at the top, summarizing the commit log since the last tag, grouped by theme (features / fixes / docs). Write the *why*, not a raw commit list.

---

## 🚀 Phase 3: Background Service Automation
Execute the release sequence — order matters.

1. **Run the FULL test suite** with the project's interpreter (e.g. `.venv/bin/python -m pytest -q`). Red suite → STOP, fix or abort. Never tag red.
2. **Commit** all bumped files together, matching the repo's release-commit style (e.g. `vX.Y.Z — <one-line theme>`).
3. **Tag**: `git tag vX.Y.Z`.
4. **Push with the tag**: `git push origin main --follow-tags`. If the human has not already authorized pushing, confirm first — a pushed tag is public.

---

## 🩺 Phase 4: Health Check & Verification

1. **All sync points agree** — re-run the Phase 1 greps; every hit must show the new version. If a version-sync test exists, run it directly.
2. **Tag is on HEAD**: `git describe --tags` prints exactly `vX.Y.Z`.
3. **Remote has the tag**: `git ls-remote --tags origin | grep vX.Y.Z` returns one line.
4. **Suite is green at the tagged commit** (already proven in Phase 3 if nothing was committed after).

---

## 🛠️ Phase 5: Automated Troubleshooting

> [!CAUTION]
> **Safety First**: Deleting or moving a tag that has already been pushed rewrites public history and breaks downstream consumers. You **MUST** get explicit human confirmation before touching a published tag.

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| **Desync found AFTER the tag was pushed** | A sync point was missed during the bump (the motivating incident). | 1. Back-fill: bump the remaining files on `main`, add a CHANGELOG note that the entry was back-filled post-tag. <br>2. Do NOT move or re-create the published tag. |
| **Tests fail mid-release** | The release was attempted on a broken tree. | Abort before tagging. Fix on `main` first; restart from Phase 1. |
| **Tagged the wrong commit (tag NOT pushed)** | Tag created before the release commit. | `git tag -d vX.Y.Z && git tag vX.Y.Z` — safe while local-only. |
| **Tagged the wrong commit (tag pushed)** | Same, discovered late. | Prefer shipping a follow-up patch release over deleting the remote tag. Remote tag deletion requires explicit human confirmation. |

---

## 🚫 Out of Scope
- Registry publishing (PyPI, npm) and CI release pipelines.
- GitHub Releases notes and artifact signing.
- Release/hotfix branch models (see `gitflow-runbook.md` only if the repo actually uses one).
