# periscope

A runbook for driving a real CDP-speaking browser (Chrome, Edge) from a coding agent — locally or across an SSH tunnel — using Google's [`chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp).

## What you get

`chrome-devtools-mcp` exposes ~45 MCP tools — navigate, click, evaluate, screenshot, console, network, lighthouse, heap snapshots, perf traces, screencasts. Full list: <https://github.com/ChromeDevTools/chrome-devtools-mcp#tools>.

You attach it to a browser the user has already launched. The MCP never SSHes, never launches a remote process, never escalates. All privileged steps are commands the user runs.

## Prerequisites

- Node 20.19+ LTS (`npx` resolves it)
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

Since Chrome 136 (April 2025), `--user-data-dir` is **mandatory** alongside `--remote-debugging-port`, even if no other Chrome is running. Without it, Chrome silently ignores the debug flag — this is post-App-Bound-Encryption malware mitigation. ([Google's announcement](https://developer.chrome.com/blog/remote-debugging-port)) Use any path that is *not* your normal profile directory. Chrome for Testing is exempt and can be installed via `npx @puppeteer/browsers install chrome@stable` if you'd rather use that.

## Scenario 2 — Local Edge (unofficial, but works)

```bash
microsoft-edge \
  --remote-debugging-port=9223 \
  --user-data-dir=/tmp/edge-cdp-profile \
  --no-first-run --no-default-browser-check \
  >/tmp/edge-cdp.log 2>&1 &
```

Then point the MCP at port 9223: `--browser-url=http://127.0.0.1:9223`.

Edge is **not officially supported** by chrome-devtools-mcp — PR [#1229](https://github.com/ChromeDevTools/chrome-devtools-mcp/pull/1229) (which added a `--browser=edge` flag) was closed without merge. But Edge speaks CDP and `--browser-url` is protocol-only, so the core verbs work. You lose `--autoConnect`, `DevToolsActivePort` auto-discovery, and recognition of `edge://` / `edge-extension://` URL schemes.

## Scenario 3 — Remote browser over SSH tunnel

On the remote box (add `--headless=new` if the host has no display server — typical for Linux servers):

```bash
google-chrome \
  --headless=new \
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

**File transfers in Scenario 3 are not transparent.** Uploads see the *remote* filesystem; downloads land on *remote* disk. `scp` or `rsync` round-trips are on you.

**Don't** use `--remote-debugging-address=0.0.0.0` to expose the port directly. Even when it binds, you have to pair it with `--remote-allow-origins=*`, at which point any local user on the box owns the browser. The SSH tunnel is the sanctioned remote path.

## `--autoConnect` (Chrome only, optional)

If you'd rather not run a separate launch command, chrome-devtools-mcp's `--autoConnect` flag will connect to a Chrome browser the user has enabled via `chrome://inspect/#remote-debugging`. Requires Chrome M144+ and does not work with Edge. The manual `--browser-url` flow above is more portable.

## Verify (doctor checklist)

Run these on the machine where the agent's MCP runs — i.e. with the tunnel in place if you're using Scenario 3.

```bash
# 1. CDP HTTP discovery responds
curl -s http://127.0.0.1:9222/json/version | head -5

# 2. There's at least one open target
curl -s http://127.0.0.1:9222/json | head -20

# 3. Tunnel is alive (Scenario 3 only)
ss -ltnp 'sport = :9222'         # Linux
lsof -nP -iTCP:9222 -sTCP:LISTEN # macOS
```

Step 1 returning a JSON blob with `Browser` and `webSocketDebuggerUrl` is the canary. If it fails, the browser isn't listening (or the tunnel isn't up).

## Common failure modes

- **`curl /json/version` returns nothing** → Chrome ignored `--remote-debugging-port`. Common causes: missing `--user-data-dir`, another Chrome owns that profile, Chrome crashed silently after launch, or (Scenario 3) tunnel is down. Check `ps aux | grep chrome` first.
- **`SingletonLock` error on launch** → another Chrome owns that profile. Find it (`ps aux | grep chrome | grep <profile-path>`) and shut it down. Only `rm /tmp/chrome-cdp-profile/SingletonLock` if you've confirmed no process owns it.
- **`bind: address already in use` on the tunnel** → leftover SSH process. `pgrep -f "ssh.*9222:localhost:9222"` to find it, then `kill`.
- **Tunnel drops mid-session** → re-run the `ssh -N -f -L …` command. The MCP reconnects on the next call; you don't need to restart it.
- **MCP connects but `take_screenshot` returns blank / black** → Wayland or headless GPU issue. Try `--ozone-platform=x11` (Wayland) or `--use-gl=swiftshader` (headless servers).
- **Edge-internal pages (`edge://settings`) misbehave** → expected. chrome-devtools-mcp doesn't recognize the `edge://` scheme. Drive regular web pages instead.

## What's not in scope

- Firefox — different debugging protocol (Marionette/RDP), not CDP.
- Headless cluster orchestration, BaaS comparisons (Browserbase/Browserless/Steel).
- Stealth / anti-fingerprint — use a hosted service.

## License

MIT.
