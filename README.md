# periscope

A runbook for driving a real CDP-speaking browser (Chrome, Edge) from a coding agent — locally or across an SSH tunnel — using Google's [`chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp).

There's no server to build here. This repo exists to remember the setup.

## What you get

`chrome-devtools-mcp` exposes ~45 MCP tools — navigate, click, evaluate, screenshot, console, network, lighthouse, heap snapshots, perf traces, screencasts. Full list: <https://github.com/ChromeDevTools/chrome-devtools-mcp#tools>.

You attach it to a browser the user has already launched. The MCP never SSHes, never launches a remote process, never escalates. All privileged steps are commands the user runs.

## Prerequisites

- Node 18+ (`npx` resolves it)
- Google Chrome and/or Microsoft Edge installed locally and/or on the remote host
- For the remote case: SSH access to a box with the browser installed

## MCP server config

Add to your client's MCP config (e.g. `.mcp.json` in a project, or the global Claude Code config):

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

`--browser-url` tells the MCP to attach to an existing browser at that CDP endpoint instead of launching its own. Same config works for local Chrome, local Edge, and a remote browser reached through an SSH tunnel — the URL is always `http://127.0.0.1:9222` from the agent's perspective.

## Scenario 1 — Local Chrome

```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-cdp-profile \
  --no-first-run --no-default-browser-check \
  >/tmp/chrome-cdp.log 2>&1 &
```

The `--user-data-dir` is **mandatory** on Chrome 136+, even if no other Chrome is running. Without it, Chrome silently ignores `--remote-debugging-port`. ([Google's announcement](https://developer.chrome.com/blog/remote-debugging-port)) Use any path that is *not* your normal profile directory.

## Scenario 2 — Local Edge (unofficial, but works)

```bash
microsoft-edge \
  --remote-debugging-port=9223 \
  --user-data-dir=/tmp/edge-cdp-profile \
  --no-first-run --no-default-browser-check \
  >/tmp/edge-cdp.log 2>&1 &
```

Then point the MCP at port 9223: `--browser-url=http://127.0.0.1:9223`.

Edge is **not officially supported** by chrome-devtools-mcp — PR #1229 (which added a `--browser=edge` flag) was closed without merge. But Edge speaks CDP and `--browser-url` is protocol-only, so the core verbs work. What you lose: Chrome 144+'s `--autoConnect` flow, `DevToolsActivePort` auto-discovery, and recognition of `edge://` / `edge-extension://` URL schemes (a handful of tools may misbehave on Edge-internal pages).

If you only need to drive a regular web page, this is fine.

## Scenario 3 — Remote browser over SSH tunnel

On the remote box:

```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir=/tmp/chrome-cdp-profile \
  --no-first-run --no-default-browser-check \
  >/tmp/chrome-cdp.log 2>&1 &
```

On your laptop:

```bash
ssh -N -f -L 9222:localhost:9222 user@box
```

- `-L 9222:localhost:9222` forwards your local `127.0.0.1:9222` → the remote's own loopback. The browser never accepts non-SSH traffic.
- `-N` = no remote command, `-f` = background after auth.
- For flaky networks: `autossh -M 0 -f -N -L 9222:localhost:9222 user@box` auto-reconnects on drop.
- For a jump host: `ssh -J jumpbox user@box -N -f -L 9222:localhost:9222`.

**Never** use `--remote-debugging-address=0.0.0.0`. Chromium [issue 40261787](https://issues.chromium.org/issues/40261787) explicitly refuses non-loopback binding as a security stance. The SSH tunnel is the sanctioned remote path.

## The Chrome 136+ `--user-data-dir` rule

Since Chrome 136 (April 2026), `--remote-debugging-port` is silently ignored on the *default* user-data-dir as well as on already-running profiles. Always launch with `--user-data-dir=<non-default-path>`. This is malware mitigation, not a bug — see Google's [post on the change](https://developer.chrome.com/blog/remote-debugging-port). Chrome for Testing is exempt; if you'd rather use that, install it via `npx @puppeteer/browsers install chrome@stable`.

## Verify (doctor checklist)

Run these on the machine where the agent's MCP runs — i.e. with the tunnel in place if you're using Scenario 3.

```bash
# 1. CDP HTTP discovery responds
curl -s http://127.0.0.1:9222/json/version | head -5

# 2. There's at least one open target
curl -s http://127.0.0.1:9222/json | head -20

# 3. Tunnel is alive (Scenario 3 only)
ss -ltnp | grep 9222

# 4. MCP can be invoked
npx -y chrome-devtools-mcp@latest --browser-url=http://127.0.0.1:9222 --help
```

Step 1 returning a JSON blob with `Browser` and `webSocketDebuggerUrl` is the canary. If it fails, the browser isn't listening (or the tunnel isn't up).

## Common failure modes

- **`curl /json/version` returns nothing** → Chrome ignored `--remote-debugging-port`. Almost always: missing `--user-data-dir`, or another Chrome was already running on the same profile dir.
- **`SingletonLock` error on launch** → previous Chrome died uncleanly. `rm /tmp/chrome-cdp-profile/SingletonLock` and relaunch.
- **`bind: address already in use` on the tunnel** → leftover SSH session. `ss -ltnp | grep 9222` to find the PID, then `kill` it.
- **Tunnel drops mid-session** → re-run the `ssh -N -f -L …` command. The MCP reconnects on the next call; you don't need to restart it.
- **MCP connects but `take_screenshot` returns blank / black** → Wayland quirk. Launch Chrome with `--ozone-platform=x11`.
- **Edge-internal pages (`edge://settings`) misbehave** → expected. chrome-devtools-mcp doesn't recognize the `edge://` scheme. Drive regular web pages instead.
- **Chrome 144+ `--autoConnect` "doesn't work" on Edge** → it can't. autoConnect is Chrome-only. Use `--browser-url` explicitly for Edge.

## What's not in scope

- Firefox — different debugging protocol (Marionette/RDP), not CDP. Out of scope.
- Headless cluster orchestration, BaaS comparisons (Browserbase/Browserless/Steel) — see `docs/` if/when it exists.
- Stealth / anti-fingerprint — use a hosted service.

## License

MIT. The runbook is yours to copy.
