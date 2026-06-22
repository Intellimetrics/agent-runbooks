# Agent Runbook - Hermes Tweet Integration

This document contains executable instructions for agentic coding tools to install, configure, and verify [Hermes Tweet](https://github.com/Xquik-dev/hermes-tweet) as a native [Hermes Agent](https://github.com/NousResearch/hermes-agent) plugin for X/Twitter workflows.

---

## What You Get
After completing this setup, Hermes gains a dedicated `hermes-tweet` toolset for:
- **Catalog discovery**: Search available X/Twitter routes through `tweet_explore` without network access.
- **Read workflows**: Use `tweet_read` for configured read-only account, search, trend, monitor, media, and evidence-gathering calls.
- **Approval-gated actions**: Keep posting, replies, likes, follows, DMs, media changes, monitors, and webhook changes disabled until `HERMES_TWEET_ENABLE_ACTIONS=true` is set for an explicitly approved session.

---

## Target Audience (Agents Only)
> [!NOTE]
> This runbook is structured for LLM-based CLI agents. When asked to add X/Twitter capabilities to Hermes Agent, execute the phases below in sequence.

> [!IMPORTANT]
> Treat `XQUIK_API_KEY` as a secret. Never paste it into prompts, issue bodies, pull request comments, logs, or tool-call arguments. Store it only in the Hermes runtime environment or `~/.hermes/.env`.

---

## Phase 1: Environment Discovery
Before installing, diagnose the host that will execute Hermes plugin code.

1. **Verify Hermes is installed**:
   - Command: `which hermes`
   - Expect: A Hermes CLI path.
2. **Check Hermes plugin state**:
   - Command: `hermes plugins list`
   - Expect: A plugin list. If `hermes-tweet` already appears, continue with verification instead of reinstalling.
3. **Find the Hermes Python environment**:
   - Command: `ls -la ~/.hermes/hermes-agent/venv/bin/python`
   - Expect: A Python interpreter path. If missing, use `hermes plugins install` in Phase 2.
4. **Check whether the API key is present without printing it**:
   - Command: `test -n "${XQUIK_API_KEY:-}" && echo "XQUIK_API_KEY present" || echo "XQUIK_API_KEY missing"`
   - Expect: Present for `tweet_read`; missing is acceptable for `tweet_explore`.
5. **Check action gate state**:
   - Command: `test "${HERMES_TWEET_ENABLE_ACTIONS:-false}" = "true" && echo "actions enabled" || echo "actions disabled"`
   - Expect: `actions disabled` for default setup.

---

## Phase 2: Configuration Injection
Install Hermes Tweet through Hermes Agent's native plugin installer first. It enables plugin metadata, prompts for required environment values in interactive sessions, and registers the plugin toolset.

```bash
hermes plugins install Xquik-dev/hermes-tweet --enable
```

If the installer cannot prompt for secrets, set the key on the runtime host outside chat, then reload or restart Hermes sessions:

```bash
mkdir -p ~/.hermes
touch ~/.hermes/.env
grep -q '^HERMES_TWEET_ENABLE_ACTIONS=' ~/.hermes/.env || printf '\nHERMES_TWEET_ENABLE_ACTIONS=false\n' >> ~/.hermes/.env
```

Only when the human operator explicitly approves storing the key in `~/.hermes/.env`, collect it without echoing:

```bash
read -r -s -p "XQUIK_API_KEY: " XQUIK_API_KEY_INPUT
printf '\nXQUIK_API_KEY=%s\n' "$XQUIK_API_KEY_INPUT" >> ~/.hermes/.env
unset XQUIK_API_KEY_INPUT
printf '\n'
```

Alternative package install path for hosts where Hermes plugin install is unavailable:

```bash
uv pip install --python ~/.hermes/hermes-agent/venv/bin/python hermes-tweet
hermes plugins enable hermes-tweet
```

---

## Phase 3: Session Activation

1. **Enable the plugin if the install command did not already enable it**:
   - Command: `hermes plugins enable hermes-tweet`
   - Expect: The plugin is enabled.
2. **Reload active interactive sessions after editing `~/.hermes/.env`**:
   - Command inside Hermes CLI or TUI: `/reload`
   - Expect: The active session picks up environment changes.
3. **Restart non-interactive gateway or cron sessions**:
   - Step: Restart the process that runs Hermes tool calls.
   - Expect: The runtime host sees `XQUIK_API_KEY` and `HERMES_TWEET_ENABLE_ACTIONS`.

---

## Phase 4: Health Check & Verification

1. **Verify the plugin is installed and enabled**:
   - Command: `hermes plugins list`
   - Expect: `hermes-tweet` appears and is enabled.
2. **Verify the toolset is visible**:
   - Command: `hermes tools list | grep -E 'hermes-tweet|tweet_'`
   - Expect: The Hermes Tweet toolset appears.
3. **Verify safe catalog discovery without network access**:
   ```bash
   hermes -z "Use tweet_explore to find X search and trend routes. Do not call tweet_read or tweet_action." --toolsets hermes-tweet
   ```
   - Expect: A catalog-oriented answer. This works without `XQUIK_API_KEY`.
4. **Verify read access only when `XQUIK_API_KEY` is configured**:
   ```bash
   hermes -z "Use tweet_explore, then call tweet_read on /api/v1/account. Do not call tweet_action." --toolsets hermes-tweet
   ```
   - Expect: Account read output or a clear API-key error.
5. **Verify action tools stay gated by default**:
   - Command: `test "${HERMES_TWEET_ENABLE_ACTIONS:-false}" = "true" && echo "actions enabled" || echo "actions disabled"`
   - Expect: `actions disabled`. Do not enable actions during setup smoke tests.

---

## Phase 5: Automated Troubleshooting

| Issue / Symptom | Root Cause | Automated Resolution Command / Step |
| :--- | :--- | :--- |
| `hermes-tweet` is installed but not visible | Third-party plugins are opt-in. | Run `hermes plugins enable hermes-tweet`, then `hermes plugins list`. |
| `tweet_explore` works but `tweet_read` fails | `XQUIK_API_KEY` is missing from the runtime host. | Set the key in the process environment or `~/.hermes/.env`, then reload interactive sessions or restart gateway sessions. |
| `tweet_action` is hidden or disabled | The default action gate is closed. | Keep it disabled unless the human explicitly approves an action session. For approved sessions only, set `HERMES_TWEET_ENABLE_ACTIONS=true` on the runtime host. |
| Desktop profile cannot use the plugin | The remote gateway host, not the desktop client, executes plugin code. | Install Hermes Tweet and set runtime environment variables on the gateway host. |
| Non-interactive install skipped the secret prompt | Hermes cannot safely prompt without a TTY. | Configure `XQUIK_API_KEY` through the runtime environment or `~/.hermes/.env`, then reload or restart. |

---

## Out of Scope
- Posting, replying, liking, following, sending DMs, or changing monitors without explicit human approval.
- Storing API keys in prompts, issue bodies, pull request comments, logs, screenshots, or tool-call JSON.
- Installing non-Hermes X/Twitter automation plugins.
- Replacing target-specific compliance, brand, or legal review workflows.
