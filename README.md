<p align="center"><a href="https://letsship.ai" target="_blank" rel="noopener"><img src="assets/ship.png" alt="SHIP" width="96" height="96"></a></p>

# SHIP skills

[![skills.sh](https://img.shields.io/badge/skills.sh-letsship%2Fskills-e84826)](https://skills.sh)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-0f0a07)](#install)
[![Cursor plugin](https://img.shields.io/badge/Cursor-plugin-0f0a07)](#install)
[![License: MIT](https://img.shields.io/badge/license-MIT-0f0a07)](LICENSE)

Teach your coding agent to hand work to SHIP. With these skills, your agent can delegate a plan or an open pull request to SHIP's autonomous coding agents. It can also follow each mission, and manage the MCP servers and sandboxes that SHIP's agents use.

SHIP runs an agentic SDLC as a managed AI software factory. Its agents plan, build, review and test each change in your own GitHub repository. Each mission ends with a pull request that passed code review and tests.

Each role runs the coding agent and the model that you choose: Claude Code, Codex, OpenCode, Kilo Code or the pi coding agent.

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

**Cursor plugin.** The same folder is also a Cursor plugin. It adds the skills and the MCP server.

**GitHub Copilot CLI and VS Code.** The folder is an Agent Plugins 1.0 package (`plugin.json`, `skills/`, `mcp.json`):

```bash
copilot plugin marketplace add letsship/skills
copilot plugin install ship@letsship
```

In VS Code, run "Chat: Install Plugin From Source". Then enter `https://github.com/letsship/skills`.

**Codex.**

```bash
codex plugin marketplace add letsship/skills
codex plugin add ship@letsship
```

**Gemini CLI.** `gemini-extension.json` adds the SHIP MCP server:

```bash
gemini extensions install https://github.com/letsship/skills
```

**Kiro.** Select "Import power from GitHub". Then enter `https://github.com/letsship/skills`.

**The SHIP CLI.** `ship init` writes the skills into `.claude/skills/` in the repository where you run it:

```bash
npm install -g @lets-ship/cli
ship init
```

## Skills

| Skill | When to use it | What it does |
| --- | --- | --- |
| [`ship-setup`](skills/ship-setup/SKILL.md) | Use when the user wants to set up or install SHIP in this repository, or when another SHIP skill says to run the setup first. | It installs the ship CLI, writes the SHIP skills into the repository and connects them to a SHIP token. |
| [`ship-delegate`](skills/ship-delegate/SKILL.md) | Use when the user wants to hand a plan, a spec or an open pull request to SHIP. | It hands work to the SHIP AI agent team from this repository. |
| [`ship-missions`](skills/ship-missions/SKILL.md) | Use when the user wants to follow a SHIP mission, check its status, or pass it feedback, review comments or a decision while it runs. | It follows, controls and finishes the SHIP missions that you or the user started. |
| [`ship-agent-runs`](skills/ship-agent-runs/SKILL.md) | Use when the user wants to read an agent's log, find why a run failed, follow a run live or stop an agent. | It reads and controls the agent runs of SHIP missions, and each run is one agent in its own sandbox. |
| [`ship-scheduled-tasks`](skills/ship-scheduled-tasks/SKILL.md) | Use when the user wants to schedule a task that SHIP runs every day, week or month, such as a dependency update. | It runs SHIP missions on a schedule. |
| [`ship-mcp-servers`](skills/ship-mcp-servers/SKILL.md) | Use when the user wants the SHIP agents to use an MCP server, or asks which MCP servers the agents can use. | It lets the SHIP agents use the MCP servers that this repository declares. |
| [`ship-sandboxes`](skills/ship-sandboxes/SKILL.md) | Use when the SHIP agents need a runtime or a tool that is not in their sandbox, or a mission waits for an agent slot. | It manages the sandboxes that the SHIP agents run in. |

## What the skills need

The skills run the `ship` CLI with a `SHIP_API_TOKEN`. If the CLI or the token is not available, your agent runs `ship-setup` first. `ship-setup` installs the CLI, writes the skills and connects them to a token. Create the token at https://letsship.ai/settings/tokens. Bind it to one project.

Then ask your agent in plain words, for example:

- "Delegate this plan to SHIP."
- "Hand this branch to SHIP for review and test."
- "What did the Reviewer say on my last mission?"

## Updates

These files come from the source code of the SHIP platform, and each release of the `ship` CLI can change them. A copy that `ship init` wrote has the CLI version in its frontmatter. When the CLI is newer, the skill tells your agent to run `ship init --force`. The copies in this repository and on the website have no version, so your agent runs `ship-setup` and installs the latest CLI.

## Security

The skills run only the `ship` CLI and read only the files of the repository that you work in. This repository contains no credentials. Report a security problem to support [at] letsship.ai.

## Privacy and support

- Privacy policy: https://letsship.ai/privacy
- Terms: https://letsship.ai/terms
- Support: https://letsship.ai/support, or support [at] letsship.ai
- Docs: https://letsship.ai/docs/mcp

## Learn more

- [What an agentic SDLC is, from issue to verified pull request](https://letsship.ai/blog/agentic-sdlc)
- [What an AI software factory is, and how to build one](https://letsship.ai/blog/what-is-an-ai-software-factory)
- [SWE-in-a-team, a coding agent benchmark of 260 missions](https://letsship.ai/blog/swe-in-a-team)
- [Run the pi coding agent headless](https://letsship.ai/blog/run-pi-coding-agent-headless)
- [Send a pull request that your coding agent wrote for review and test](https://letsship.ai/platform/agent-to-pr)

## License

The files in this repository are under the MIT license. The SHIP service is under the [SHIP terms](https://letsship.ai/terms).
