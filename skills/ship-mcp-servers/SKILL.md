---
name: ship-mcp-servers
description: Let the SHIP agents use the MCP servers that this repository declares. Bind a server to agent roles with a credential, approve its tools, start an OAuth sign-in, choose the MCP file or remove a binding. Use when the user says "let the SHIP agents use our GitHub MCP server" or "give the planner Linear access". Also use it when the user asks which MCP servers the agents can use.
---

# Give the agents MCP servers

The SHIP agents can use the MCP servers that the repository declares in its MCP file, which is `.mcp.json` by default.
SHIP never loads that file on its own. A server reaches the agents only through a binding. A binding names the agent
roles that may use the server, the tools that they may call, and the credential that SHIP attaches. The agents never
see the credential.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship mcp list`, `ship mcp bindings` | `connectors:read` |
| `ship mcp tools` without flags | `connectors:read` |
| `ship mcp bind`, `ship mcp connect`, `ship mcp remove`, `ship mcp file` | `connectors:write` |
| `ship mcp tools --refresh`, `ship mcp tools --approve` | `connectors:write` |

Changes also need the admin or owner role in the organization. A token never has more access than the person who
created it.

## See what the repository declares

```bash
ship mcp list --project prj_123
ship mcp bindings
```

`ship mcp list` shows the servers in the MCP file of one project and the binding of each. `ship mcp bindings` shows
every binding in the projects of your organization. Neither command shows a credential value. They show only the names
of the fields that are set.

## Bind a server

`--roles` lists every agent that may use the server: `planner`, `builder`, `reviewer` and `qa`, or `none`. The Reviewer
and the QA agent get only the tools that read, and they cannot use a server that runs inside the sandbox.

```bash
ship mcp bind github --roles planner,qa --header-file Authorization=gh.txt
ship mcp bind sentry --roles qa --env-var-file KEY=t --allow-host sentry.io
ship mcp bind linear --roles planner,builder --no-credential
```

Give credentials as files, so that they stay out of the shell history. Do not put a token in a command line.

- `--header-file <Name=path>`: a request header for a remote server. The file holds the complete value, for example
  `Bearer <token>`. You can give the flag more than one time.
- `--env-var-file <NAME=path>` with `--allow-host <host>`: an environment variable for an isolated server, and the hosts
  that the credential may go to. These flags imply `--transport isolate`.
- `--no-credential`: bind with no credential, and remove a stored one.

Without a credential flag, the binding keeps the credential that it has.

`--transport` is `remote`, `isolate` or `local`. With `remote`, SHIP sends requests to the URL of the server. With
`isolate`, SHIP runs the npm package of the server in isolation, so that the server can hold a secret. With `local`, the
server runs inside the sandbox with no credential. The default is what the declaration of the server allows. `--url`
sets a fixed https URL for the server. Without `--transport` or `--url`, the binding keeps its current server.

## Approve the tools

A new binding gives the agents no tools. Approve them after you bind the server:

```bash
ship mcp tools github --refresh
ship mcp tools github --approve get_issue,get_file --attest get_file
```

Without flags, the command prints the last tool list of the server, with the approval of each tool and whether it only
reads. The `--refresh` flag gets the current list from the server first. The `--approve` flag replaces the approved set
with the tools that you name. A tool that has no annotation from the server goes to the Reviewer and the QA agent only if you also
name it in `--attest`. With `--attest`, you confirm that the tool only reads. Name only the tools that the task needs.

## Sign in to a server that uses OAuth

```bash
ship mcp connect linear --project prj_123
```

The command prints a link. A person opens it in a browser where they are signed in to the SHIP console. The provider
then sends them back to the console, which stores the credential. You cannot complete this step for the user. Give the
user the link. The link works one time and expires.

## Remove a binding or change the MCP file

```bash
ship mcp remove notion --project prj_123
ship mcp file .cursor/mcp.json --project prj_123
ship mcp file default
```

`ship mcp remove` deletes the binding and its stored credential. From their next run, the agents do not get the server.
The command does not change the MCP file of the repository.

`ship mcp file` sets the file that declares the servers of the project, for example `.cursor/mcp.json`. The value
`default` sets it back to `.mcp.json`. SHIP refuses the command while the `ship.yml` of the repository sets
`mcp.config`, because that setting has priority.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have the scope of the command, or the role cannot change it. | Ask the user for a token with the scope, from an admin. Do not retry. |
| 4 | SHIP found no such server, binding or project. | Check the server name with `ship mcp list`, or give `--project <id>`. |
| 5 | The change needs something that is not true yet. | Do what the message says, then run the command again. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
