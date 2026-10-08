<p align="center"><img src="assets/ship.png" alt="SHIP" width="96" height="96"></p>

# SHIP skills

[![skills.sh](https://img.shields.io/badge/skills.sh-letsship%2Fskills-e84826)](https://skills.sh)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-0f0a07)](#install)
[![Cursor plugin](https://img.shields.io/badge/Cursor-plugin-0f0a07)](#install)
[![License: MIT](https://img.shields.io/badge/license-MIT-0f0a07)](LICENSE)

Teach your coding agent to hand work to SHIP. The skills delegate a plan or an open pull request to SHIP's agent fleet, follow each mission, and manage the MCP servers and sandboxes that the fleet uses. SHIP's agents plan, build, review and test each change in your own GitHub repository.

## Install

**Any coding agent, with the skills CLI.** It detects your agents and installs the skills for each one.

```bash
npx skills add letsship/skills
```

To pick one skill, or to see the list first:

```bash
npx skills add letsship/skills --skill ship-delegate
npx skills add letsship/skills --list
```

The same skills are on the SHIP website, so `npx skills add https://letsship.ai` works too.

**Claude Code plugin.** The plugin adds the skills and the hosted SHIP MCP server:

```text
/plugin marketplace add letsship/skills
/plugin install ship@letsship
```

**Cursor plugin.** The same folder is a Cursor plugin, with the skills and the MCP server.

**GitHub Copilot CLI and VS Code.** The folder is an Agent Plugins 1.0 package (`plugin.json`, `skills/`, `mcp.json`):

```bash
copilot plugin marketplace add letsship/skills
copilot plugin install ship@letsship
```

In VS Code, run "Chat: Install Plugin From Source" and enter `https://github.com/letsship/skills`.

**Codex.**

```bash
codex plugin marketplace add letsship/skills
codex plugin add ship@letsship
```

**Gemini CLI.** `gemini-extension.json` adds the SHIP MCP server:

```bash
gemini extensions install https://github.com/letsship/skills
```

**Kiro.** Select "Import power from GitHub" and enter `https://github.com/letsship/skills`.

**The SHIP CLI.** `ship init` writes the skills into `.claude/skills/` in the repository you run it in:

```bash
npm install -g @lets-ship/cli
ship init
```

## Skills

| Skill | What it does |
| --- | --- |
| [`ship-setup`](skills/ship-setup/SKILL.md) | Set up SHIP in this repository. |
| [`ship-delegate`](skills/ship-delegate/SKILL.md) | Hand work to the SHIP agent fleet from this repository. |
| [`ship-missions`](skills/ship-missions/SKILL.md) | Follow and steer the SHIP missions that you or the user already delegated. |
| [`ship-mcp-servers`](skills/ship-mcp-servers/SKILL.md) | Let the SHIP agents use the MCP servers that this repository declares. |
| [`ship-sandboxes`](skills/ship-sandboxes/SKILL.md) | Manage the sandboxes that the SHIP agents run in. |

## What the skills need

The skills run the `ship` CLI with a `SHIP_API_TOKEN`. If either is missing, your agent runs `ship-setup` first: it installs the CLI, writes the skills and connects them to a token. Create the token at https://letsship.ai/settings/tokens, and bind it to one project.

Then ask your agent in plain words, for example:

- "Delegate this plan to SHIP."
- "Hand this branch to SHIP for review and test."
- "What did the Reviewer say on my last mission?"

## Updates

These files come from the SHIP platform's own source, and each release of the `ship` CLI can change them. A copy that `ship init` wrote carries the CLI version in its frontmatter, and the skill tells your agent to run `ship init --force` when the CLI is newer. The copies in this repository and on the website carry no version, so your agent runs `ship-setup` and installs the latest CLI.

## Security

The skills run only the `ship` CLI and read only the files of the repository you work in. This repository holds no credentials. Report a security problem to support@letsship.ai.

## Privacy and support

- Privacy policy: https://letsship.ai/privacy
- Terms: https://letsship.ai/terms
- Support: support@letsship.ai
- Docs: https://letsship.ai/docs/mcp

## License

MIT. The SHIP service is under the [SHIP terms](https://letsship.ai/terms).
