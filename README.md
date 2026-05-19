# agent-runbooks

Portable markdown runbooks for setting up coding-agent tooling. No code, no skill format — just markdown. Works in Claude Code, codex, Gemini CLI, GitHub Copilot CLI, or read straight off disk.

## Runbooks

- [chrome-devtools-mcp](chrome-devtools-mcp.md) — drive a CDP-speaking browser (Chrome, Edge) from a coding agent, locally or over an SSH tunnel.

## Why runbooks, not skills

Skills (the Claude-Code-specific kind invoked via the `Skill` tool, living under `~/.claude/skills/`) only work in Claude Code. These runbooks are plain markdown so they're portable across every coding-agent CLI worth using.

If you ever want a Claude-Code-skill version of one of these, the skill can `Read` the runbook rather than duplicating its content.

## Conventions for new runbooks

- One topic per file at the repo root: `<topic>.md`.
- Add a one-line entry to this index when you add a runbook.
- Terse. Commands first, prose only where the *why* is non-obvious.
- Include a doctor checklist (`curl`/`ss`/`lsof` etc.) and a "common failure modes" section. Both are recurring readers' needs.
- Audit fresh runbooks against the llm-council (`council_run` with `mode=review`) before considering them done — round 1 catches factual errors, round 2 catches precision issues.

## License

MIT.
