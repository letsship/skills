---
name: ship-setup
description: Set up SHIP in this repository. It installs the ship CLI, writes the SHIP skills into the repository and connects them to a SHIP token. Use when the user says "set up https://letsship.ai/SKILL.md", "install SHIP" or "connect this repo to SHIP". Also use it when another SHIP skill tells you to run the setup first.
---

# Set up SHIP

SHIP runs a fleet of agents that plan, build, review and test changes to a repository. The `ship` CLI is how you
hand work to that fleet. This document installs the CLI and the skills that teach you each SHIP capability. Do these
steps one time in each repository, and again when the CLI has a new version.

## 1. Install or upgrade the CLI

This document is the hosted copy, so always install the latest CLI. The same command installs the CLI or upgrades it.

```bash
npm install -g @lets-ship/cli@latest
ship --version
```

Do not install an older version to match an old skill copy. Upgrade the CLI and write the skills again.

## 2. Write the skills into the repository

```bash
ship init          # --force replaces copies that are stale or edited
```

The command writes one skill for each SHIP capability into `.claude/skills/`. The `version` in the frontmatter of each
copy is the CLI version that wrote it. Later sessions read the local copies and do not fetch them again.

| Skill | Use it to |
| --- | --- |
| `ship-delegate` | Hand a plan, a spec or a finished branch to SHIP. |
| `ship-missions` | Read the state of a mission and send it instructions. |
| `ship-mcp-servers` | Let the agents use the MCP servers that the repository declares. |
| `ship-sandboxes` | Add runtimes and tools to the sandbox that the agents run in, and read the agent capacity. |

If a copy already exists, `ship init` writes nothing and names the file. If the `version` of a copy is older than
`ship --version`, run `ship init --force`. That replaces every copy, and edits to the copies are lost.

The same skills are in the discovery index at <https://letsship.ai/.well-known/agent-skills/index.json>. An agent that
reads skills from the web can fetch them from there.

## 3. Get a token

`SHIP_API_TOKEN` is a personal access token. A person creates it in the SHIP console at
<https://letsship.ai/settings/tokens>. You cannot create a token yourself, because the API refuses a token request that
comes from another token. Ask the user to create a token and export it:

```bash
export SHIP_API_TOKEN=pat_...
```

Each skill names the scopes that its commands need. Ask for the scopes of the skills that the user wants, and no more.
The user can also limit the token to one project. Then the token can only reach the missions, runs, environments,
artifacts and scheduled tasks of that project. Suggest a project token when the work is about one project.

For the complete rules about SHIP credentials, read <https://letsship.ai/auth.md>.

## 4. Check the token

```bash
curl -fsS -H "Authorization: Bearer $SHIP_API_TOKEN" "${SHIP_API_BASE_URL:-https://api.letsship.ai/v1}/me"
```

The response gives the user, the organization, the role and the scopes of the token. If the request fails with `401`,
the token is not valid. Ask the user for a new token.

## 5. Find the project

`ship` finds the project from the git remote of the repository. If the remote matches no project, or more than one
project, the CLI tells you. Then give `--project <id>`, or export `SHIP_PROJECT_ID`. Do not guess a project id.

## 6. Choose the platform

`ship` talks to `https://api.letsship.ai/v1`, the production API. For a staging or self-hosted platform, set
`SHIP_API_BASE_URL`. The console links that the CLI prints then point to the same platform.

## When something fails

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 3 | The token is not set, or it does not have the scope that the command needs. | Ask the user for a token with that scope. Do not retry with the same token. |
| 6 | The organization did not finish the onboarding in the console. | Ask the user to finish the onboarding. |
