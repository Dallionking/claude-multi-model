# claude-multi-model

**Run GPT and Grok inside Claude Code — one harness, every brain, zero API bills.**

Your Claude Code setup (skills, hooks, subagents, MCP servers) is the best agent
harness you own. This skill teaches YOUR Claude to install
[CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) on your machine,
sign in with the subscriptions you already pay for (ChatGPT/Codex, X Premium/Grok),
and mint one-word launchers:

```bash
claude          # Anthropic models, as always
claude-gpt      # same harness, GPT brain
claude-grok     # same harness, Grok brain
```

## Install (30 seconds)

```bash
mkdir -p ~/.claude/skills/multi-model-setup
curl -fsSL https://raw.githubusercontent.com/Dallionking/claude-multi-model/main/SKILL.md \
  -o ~/.claude/skills/multi-model-setup/SKILL.md
```

Then open Claude Code and say:

> set up cli proxy so I can run gpt and grok in claude code

Claude walks you through the whole thing — install, account OAuth, launchers,
verification — and teaches you the four gotchas that bite everyone (the `/model`
Enter trap is real).

## What you need

- Claude Code installed
- At least one of: a ChatGPT subscription (GPT models) or X Premium (Grok models)
- macOS or Linux

## Multi-agent bonus

If you run [Orca](https://orca.dev) or any terminal-cockpit workflow, the skill
includes the dispatch recipe for using these launchers as parallel worker seats,
plus a starter model-lane table for your `AGENTS.md`.

## Credits

Built on [CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) by
router-for-me. This repo is just the Claude-native setup skill.

MIT license.
