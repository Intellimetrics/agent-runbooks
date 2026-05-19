# periscope

An MCP server that lets coding agents drive a real browser — Chrome, Edge, Brave, Chromium — over the Chrome DevTools Protocol (CDP), whether the browser is running locally or on a remote machine reached through an SSH tunnel.

**Status:** initialized, not yet built. See [CLAUDE.md](CLAUDE.md) for the handoff brief.

## Why this exists

Coding agents often need to inspect or control a real browser:

- DOM and computed styles, not just the HTML source
- Console logs and uncaught errors
- Network requests, including XHR/fetch payloads
- Click handlers, JS-driven UI, single-page-app routing
- State that only exists after auth, after cookies, or after a long-running JS app boots

The existing options each have a gap. Anthropic's `claude-in-chrome` MCP requires a Chrome extension and a logged-in Claude account — it can't reach a headless Chrome on a remote box. Playwright/Puppeteer scripts are powerful but require the agent to write fresh code for every interaction. Raw CDP over `curl` and `wscat` works but is tedious.

`periscope` is the missing piece: a single MCP that exposes a small, stable surface (`navigate`, `click`, `evaluate`, `screenshot`, `read_console`, `read_network`, …) and connects to whichever CDP-speaking browser the user points it at — local or remote.

## Core design constraints

1. **Multiple browsers.** Chrome, Edge, Brave, Chromium, anything that speaks CDP. Firefox via Marionette is out of scope for v0.
2. **Local and remote.** Local browsers are the easy case (`http://localhost:9222`). Remote browsers go through an SSH tunnel (`ssh -L 9222:localhost:9222 user@host`).
3. **The MCP never SSHes on its own.** When the user needs to set up a remote tunnel or launch a browser on a remote machine, the MCP returns the exact shell command for the agent to surface to the user. The user runs it, then tells the agent to retry. The MCP is the orchestrator, not the executor of privileged commands.
4. **Stateless reconnection.** If the tunnel drops, the MCP should reconnect cleanly to the same target on the next call. No long-lived assumptions about page state.
5. **Read-by-default.** Mirror the llm-council convention: read tools (inspect, screenshot, read_console) are always safe; write tools (click, type, evaluate) are clearly named and ideally gated.

## Reference

Built in the same shape as [llm-council](https://github.com/Intellimetrics/llm-council) — Python 3.11+, MCP server + CLI, MIT licensed, published to the Intellimetrics GitHub org.

Local path for the reference project: `/development/projects/active/llm-council/`
