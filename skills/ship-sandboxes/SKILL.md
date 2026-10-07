---
name: ship-sandboxes
description: Manage the sandboxes that the SHIP agents run in. Add runtimes and tools to a custom sandbox, build it, read its build log and roll it back. Make a sandbox the default of a project, and connect private package registries. Also read how many agents your missions run at the same time. Use when the user says "the agents need Python", "add a JDK to the SHIP sandbox" or "why does my mission wait".
---

# Manage the sandboxes that agents run in

Every SHIP agent runs in a sandbox. The platform base has what the agents need to run. A custom sandbox adds
steps to that base, for the runtimes and tools that a repository needs. Each change to a sandbox is a new version, and
missions start from the active version.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship sandbox list`, `ship sandbox starters`, `ship sandbox versions`, `ship sandbox logs` | `agents:read` |
| `ship sandbox create`, `ship sandbox update`, `ship sandbox build`, `ship sandbox rollback`, `ship sandbox delete` | `agents:read` and `agents:write` |
| `ship sandbox default` | `projects:admin` |
| `ship sandbox registry list` | `connectors:read` |
| `ship sandbox registry set`, `ship sandbox registry remove` | `connectors:write` |
| `ship sandbox capacity` | `sandboxes:read` |

Changes also need the admin or owner role in the organization. A token never has more access than the person who
created it.

## See the sandboxes and starters

```bash
ship sandbox list
ship sandbox starters
```

The `ship sandbox list` command shows each custom sandbox with its active version, its size and whether its last build
is ready. The `ship sandbox starters` command shows the starter definitions for Python, Java, Go, Rust and Ruby. Each
starter has a size that fits it.

## Create a sandbox

```bash
ship sandbox create --name python-uv --starter python --wait
ship sandbox create --name ml --file ml.Dockerfile --size 2x8192x16000
```

The Dockerfile has steps only: `RUN`, `ENV`, `ARG` and `WORKDIR`. The platform supplies the base, so the file has no
`FROM`. There is also no build context, so `COPY` cannot work. The first build starts immediately. With `--wait`, the
command shows the build log until the build finishes, and exits 7 with the reason if the build fails.

`--size` is `standard-1` to `standard-4`, or a custom `<vcpu>x<memoryMib>x<diskMb>`. A custom size has 1 to 4 vCPU and
8000 to 20000 MB of disk. Its memory is at least 3072 MiB for each vCPU, and at most 12288 MiB.

## Change, build and roll back

```bash
ship sandbox update python-uv --file sandbox.Dockerfile --wait
ship sandbox update sbt_1a2b --size standard-4
ship sandbox build python-uv --wait
ship sandbox logs python-uv --follow
ship sandbox logs sbt_1a2b --build tbd_9f8e
ship sandbox versions python-uv
ship sandbox rollback python-uv --to 2
```

A new definition or size becomes the next version. SHIP makes that version active and builds it. A mission that starts
before the build finishes waits for it, and if the build fails, the mission stops with the reason. A new `--name` alone
does not start a build. A sandbox argument can be the id or the name.

The `ship sandbox build` command builds the active version now. If a build is already in progress, the command returns
that build. The `ship sandbox logs` command prints the log of the last build, or of the build that `--build` names. The
`--follow` flag shows the log until the build finishes.

You cannot change a version, but the `ship sandbox rollback` command makes an earlier version active. If
the snapshot of that version still exists, the rollback is instant. If not, SHIP builds the version again.

## Choose the sandbox of a project

```bash
ship sandbox default python-uv --project prj_123
ship sandbox default none
```

This sets the default sandbox of the project. Three settings have priority over it: `agents.<role>.sandbox` in
`ship.yml`, a `.ship/sandbox.Dockerfile` in the repository, and the `--sandbox` flag of one hand-off (see the
`ship-delegate` skill). The value `none` sets the project back to the platform base.

## Delete a sandbox

```bash
ship sandbox delete python-uv
```

SHIP refuses to delete a sandbox that a project uses as its default. If a `ship.yml` entry still names the sandbox, the
dispatch of that role fails, and the error names the sandbox. Change that entry before you delete the sandbox.

## Connect a private package registry

```bash
ship sandbox registry set --host npm.pkg.github.com --token-file gh.txt
ship sandbox registry set --host acme.io --username ci --password-file pw
ship sandbox registry list
ship sandbox registry remove npm.pkg.github.com
```

SHIP adds the credential to the requests to that host when they leave the sandbox. The credential never enters the
sandbox, its build log or its snapshot. Give a bearer token with `--token-file`, or a user name with `--username` and
`--password-file`. Give secrets as files, so that they stay out of the shell history.

## Read the agent capacity

```bash
ship sandbox capacity
```

Your organization can run as many agents at the same time as its plan allows. If a new agent would go over that number,
its mission waits until one of your agents finishes. A mission can also wait when all agent slots on the platform are in
use. Use this command when the user asks why a mission waits.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have the scope of the command, or the role cannot change it. | Ask the user for a token with the scope, from an admin. Do not retry. |
| 4 | SHIP found no such sandbox, build or project. | Check the name with `ship sandbox list`, or give `--project <id>`. |
| 5 | The change needs something that is not true yet, for example a delete of a project default. | Do what the message says, then run the command again. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
| 7 | A build that the command waited for failed. The message is its reason. | Fix the Dockerfile, then update the sandbox again. |
