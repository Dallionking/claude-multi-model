---
name: multi-model-setup
description: >
  Walk the user through running GPT, Grok, and other models INSIDE the Claude Code
  harness via CLIProxyAPI — install, OAuth account connect (ChatGPT/Codex, X Premium/Grok),
  launcher scripts, verification, and optional Orca multi-agent dispatch. Use when the
  user says "set up cli proxy", "run gpt in claude code", "use grok in claude",
  "multi-model claude", or "one harness every model".
---

# Multi-Model Claude Code — CLIProxyAPI setup

You are walking the user through giving Claude Code extra brains: the same harness
(skills, hooks, subagents, MCP servers) driven by GPT or Grok models through their
EXISTING subscriptions — no API billing. Explain each phase in one plain sentence
before running it. Verify every phase with a real command before moving on.

**The mental model (tell the user this first):** Claude Code mails letters to
api.anthropic.com. CLIProxyAPI is a mailbox on their own machine that forwards
letters to OpenAI or xAI instead, signed in with their ChatGPT / X Premium
subscriptions. One launch word picks which brain answers.

## Phase 0 — Preflight

```bash
claude --version          # Claude Code installed?
uname -s                  # macOS (Darwin) or Linux
ls ~/.local/bin 2>/dev/null | head -3   # launcher home exists?
```
- Requires: Claude Code, plus at least one of a ChatGPT (Codex) subscription or
  X Premium (Grok) subscription. A Claude subscription stays signed in for plain `claude`.
- `mkdir -p ~/.local/bin ~/.cli-proxy-api` and ensure `~/.local/bin` is on PATH.

## Phase 1 — Install the proxy binary

Download the latest release for the user's OS/arch from
`https://github.com/router-for-me/CLIProxyAPI/releases` (asset naming:
`CLIProxyAPI_<version>_<os>_<arch>.tar.gz`; use `gh release download -R
router-for-me/CLIProxyAPI --pattern '*darwin_arm64*'` when `gh` is available,
otherwise `curl -L` the asset URL). Then:

```bash
tar -xzf CLIProxyAPI_*.tar.gz
mv cli-proxy-api ~/.local/bin/ && chmod +x ~/.local/bin/cli-proxy-api
~/.local/bin/cli-proxy-api --help | head -3   # verify it runs
```

## Phase 2 — Config

Write `~/.cli-proxy-api/config.yaml`:

```yaml
host: "127.0.0.1"
port: 8317
auth-dir: "~/.cli-proxy-api"
api-keys:
  - "sk-local-<GENERATE: openssl rand -hex 16>"
debug: false
```

Generate the key with `openssl rand -hex 16` — this is a LOCAL password between
Claude Code and the proxy, not a vendor API key. `chmod 600` the file.

## Phase 3 — Connect accounts (OAuth, one browser tab each)

Run each login the user has a subscription for. Each opens a browser, the user
signs in, and a credential JSON lands in `~/.cli-proxy-api/`:

```bash
~/.local/bin/cli-proxy-api -config ~/.cli-proxy-api/config.yaml -codex-login   # ChatGPT/Codex sub → GPT models
~/.local/bin/cli-proxy-api -config ~/.cli-proxy-api/config.yaml -xai-login     # X Premium → Grok models
# also available: -claude-login, -kimi-login, -antigravity-login
```

Headless/SSH box: add `-no-browser` and open the printed URL manually.

## Phase 4 — Run the daemon + verify

```bash
~/.local/bin/cli-proxy-api -config ~/.cli-proxy-api/config.yaml > ~/.cli-proxy-api/proxy.log 2>&1 &
sleep 2
curl -s -H "Authorization: Bearer <the sk-local key>" http://127.0.0.1:8317/v1/models | head -c 400
```

The models list must show entries from every provider they logged into. If they
want it always-on, offer a launchd LaunchAgent (macOS) / systemd user unit (Linux)
that runs the same command at login.

## Phase 5 — Launcher scripts (NOT aliases — this matters)

One executable per brain, in `~/.local/bin`. Template (`claude-gpt` shown; repeat
for each model the proxy lists, e.g. `claude-grok` with `grok-4.5`):

```bash
#!/bin/bash
# claude-gpt — Claude Code harness, GPT brain, via local CLIProxyAPI
export CLIPROXY_KEY="sk-local-<their key>"
export ANTHROPIC_BASE_URL="http://127.0.0.1:8317"
export ANTHROPIC_AUTH_TOKEN="$CLIPROXY_KEY"
MODEL="${CLIPROXY_MODEL:-gpt-5.6-sol}"        # override per-launch: CLIPROXY_MODEL=<id> claude-gpt
export ANTHROPIC_MODEL="$MODEL"
export ANTHROPIC_SMALL_FAST_MODEL="$MODEL"
exec claude --model "$MODEL" "$@"
```

`chmod +x` each. **Why scripts, not aliases:** many setups wrap `claude` in a
shell function (tmux relaunch, logging). An alias whose last word is `claude`
resolves into that wrapper, which respawns the process WITHOUT your env vars —
the model flag survives, the proxy redirect is silently lost, and the session
errors with "model may not exist". A script with `exec claude` bypasses shell
functions entirely.

**Verify end-to-end (do not skip):**

```bash
claude-gpt -p "Reply with exactly: PROXY OK"
```

## The four rules to teach the user before you finish

1. **Engine is picked at launch.** The proxy address is read once at startup —
   `/model` can switch Claude tiers, never families. Different brain = new terminal.
2. **Never press Enter in `/model` inside a proxy session.** Enter saves the
   custom model as the GLOBAL default and every plain `claude` after that errors.
   Use `s` (this session only) or Esc. If it happens: fix the `"model"` key in
   `~/.claude/settings.json`.
3. **The "claude.ai connectors are disabled" banner is expected** in proxy
   sessions — custom auth takes precedence over claude.ai login. Harmless.
4. **The proxy must be running.** Quick check: `curl -s http://127.0.0.1:8317/v1/models`.

## Optional — Orca / multi-agent dispatch

If the user runs Orca (or any tmux-style agent cockpit): the launchers are
first-class worker seats. Orca's `--agent` flag only knows built-in ids, so
launch proxy seats by command:

```bash
orca worktree create --name <task> --json                 # no --agent
orca terminal create --worktree <selector> --command "claude-gpt --dangerously-skip-permissions" --title "gpt:<task>"
orca terminal send --terminal <handle> --text "<the brief>" --enter
```

Suggested lane split to put in the project's `AGENTS.md` (adjust to taste):

| Lane | Seat |
|---|---|
| Planning, review, frontend/design taste | Claude (Opus/Sonnet via plain `claude`) |
| Hard implementation, adversarial review | `claude-gpt` (strong GPT tier) |
| Clear-spec implementation, bulk mechanical | `claude-grok` |
| Cross-model review of any diff | a DIFFERENT family than the author |

One harness, every brain — the user's skills, hooks, and MCP servers apply to
all of them identically.
