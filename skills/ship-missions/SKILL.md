---
name: ship-missions
description: Follow and steer the SHIP missions that you or the user already delegated. Read the stage, review, test and pull request of a mission, and send it instructions or feedback. Use when the user asks "what is the status of SHIP-412", "is SHIP done with it" or "tell SHIP to change X". Also use it when the user wants to pass review comments or a decision to a mission that already runs.
---

# Follow and steer a mission

A mission is the work that SHIP does for one issue. Use this skill after a hand-off (see the `ship-delegate` skill), to
read the state of a mission and to send it new information.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship status` | `missions:read` |
| `ship instruct` | `missions:write` |
| `ship run stop` | `sandboxes:stop` (an org admin's token) |

## Read the state of a mission

```bash
ship status SHIP-412
ship status SHIP-412 --json    # for jq
```

The command prints the stage of the mission, its review state, its test state and its pull request. Read it before
you tell the user that work is complete.

## Send feedback to a mission

When the user decides something about work that SHIP already has, send it as an instruction. Examples are an answer to
a question from the agents, review comments, or a correction to a choice that an agent made. The instruction goes to
the agents and starts the Builder on the open pull request of the mission.

```bash
ship instruct SHIP-412 -m "Keep the current Node range. Override nopt instead."
ship instruct SHIP-412 --file review-notes.md
ship instruct SHIP-412 -m "FYI staging is down until 14:00" --note
```

`--note` records context for the next agent and starts no work. Use it for background information. Leave it out for
anything that you want the agents to act on.

Use an instruction, not a new hand-off or a restart. A restart checks the same pull request against the review and test
results that it already has. It finds no change and correctly does nothing, so the feedback goes nowhere. An instruction
is the channel for new information.

The reply tells you what SHIP recorded and what it starts:

```
SHIP-412  instruction recorded (ins_9b8e2a3f3c4d)
  mission   ready_to_merge
  dispatch  builder: it will pick this up on the mission's open pull request
```

When SHIP starts no work, the reply says why and what to do instead. An example is a pull request that is already
merged: the remedy is a new mission, not feedback to a finished one. Read that line before you report back. Do not tell
the user that an instruction arrived until the reply says `recorded`.

## Stop one agent run

Stop a run only when the user asks for it, because the agent's work in progress is lost. Give the sandbox id of the
run. The console shows it in the Sandbox ID field of the run. The `list_sandboxes` MCP tool and `GET /v1/sandboxes`
also give it.

```bash
ship run stop builder-ship-412-3f9c41aa-coding-1
```

The status of the run becomes `stopped` a few seconds later. The command stops one run, not the mission.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have the scope of the command. | Ask the user for a token with the scope. Do not retry. |
| 4 | SHIP found no such mission, project or sandbox. | Check the issue id or the sandbox id, or give `--project <id>`. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
