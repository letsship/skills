---
name: ship-setup
description: Use when the user wants to set up or install SHIP in this repository, or when another SHIP skill says to run the setup first. It installs the ship CLI, writes the SHIP skills into the repository and connects them to a SHIP token. For example, the user says "set up https://letsship.ai/SKILL.md", "install SHIP" or "connect this repo to SHIP".
---

# Set up SHIP

SHIP runs a team of AI coding agents that plan, build, review and test changes to a repository. The `ship` CLI is how you
hand work to that agent team. This document installs the CLI and the skills that teach you each SHIP capability. Do these
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
| `ship-missions` | Read the state of a mission, send it instructions, control it, and read its preview, proof and credits. |
| `ship-agent-runs` | Read the runs of the agents of a mission, their cost and their logs, and stop one run. |
| `ship-scheduled-tasks` | Run missions on a schedule. |
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

To see the tokens of the user, or to revoke one that leaked or is no longer in use, run:

```bash
ship token list
ship token revoke tok-pat-9b8e2a3f
```

The list shows only the tokens of the person who created the current token, never a secret. A revoked token fails at
once, and nobody can make it work again. Revoke a token only when the user asks for it.

For the complete rules about SHIP credentials, read <https://letsship.ai/auth.md>.

## 4. Check the token

```bash
ship whoami
```

The command prints the user, the organization, the role and the scopes of the token. For a project token, it also
prints the project. If the command exits 3, the token is not set or not valid. Ask the user for a new token.

## 5. Find the project

```bash
ship project list
ship project get prj_123
```

`ship project list` lists the projects that the token can reach, each with its id and its repository. Find the project
of this repository, then give `--project <id>` to the commands that need it, or export `SHIP_PROJECT_ID`. If more than
one project uses the repository, ask the user which one. Do not guess a project id. `ship project get` shows the
repository, the tracker and the default sandbox of one project.

```bash
ship project update prj_123 --name web-app
ship project update prj_123 --status paused
ship project update prj_123 --auto-merge on
```

`ship project update` changes the settings of a project, and only the ones that you give. A paused or archived project
starts no new missions. With auto-merge on, SHIP merges each ready pull request of the project, and no person merges it.
The flags `--repo` and `--linear-team` connect the project to another repository or Linear team. Only an owner or an admin can
change a project, and the token needs `projects:admin`. Change a project only when the user asks for it.

## 6. Choose the platform

`ship` talks to `https://api.letsship.ai/v1`, the production API. For a staging or self-hosted platform, set
`SHIP_API_BASE_URL`. The console links that the CLI prints then point to the same platform.

## When something fails

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 3 | The token is not set, or it does not have the scope that the command needs. | Ask the user for a token with that scope. Do not retry with the same token. |
| 6 | The organization did not finish the onboarding in the console. | Ask the user to finish the onboarding. |
