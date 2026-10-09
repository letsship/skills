---
name: ship-agent-runs
description: Use when the user wants to read an agent's log, find why a run failed, follow a run live or stop an agent. It reads and controls the agent runs of SHIP missions, and each run is one agent in its own sandbox. List the runs of a mission or of the organization, and read the status, resource use and cost of one run. Read what the agent did from its log, follow one run live until it ends, and stop one run. For example, the user asks "what did the builder do on SHIP-412", "why did the QA run fail" or "show me the agent log". Also use it when the user says "stop that agent", or to find the run that makes a mission slow or expensive.
---

# Read and control agent runs

A mission is the work that SHIP does for one issue. The agents of a mission do that work in runs. A run is one agent
in its own sandbox, for example one Planner run, three Builder runs and one QA run. The id of a run is its sandbox id,
such as `builder-ship-412-3f9c41aa-coding-1`.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship run list`, `ship run get`, `ship run logs`, `ship run watch` | `sandboxes:read` |
| `ship run stop` | `sandboxes:stop` (an org admin's token) |

## When to use it

- The user asks what an agent does now, or what it did.
- You must wait for a run to end before your next step.
- A run failed, and you must find the cause before you send an instruction to the mission.
- A mission costs more or takes longer than the user expects, and you must find the run that is the cause.
- The user asks you to stop one agent.

To read the state of the mission itself, or to send it feedback, use the `ship-missions` skill.

## Find the runs of a mission

```bash
ship status SHIP-412
ship run list --mission SHIP-412
```

`ship status` lists the runs of the mission, oldest first, with the id of each run. The current run has the mark
`(current)`. `ship run list` gives the same runs with their harness and start time, newest first.

You can also find runs across the organization. A list covers every project of the token, unless `--project` names one.
`SHIP_PROJECT_ID` does not narrow a list.

```bash
ship run list --status running
ship run list --mission SHIP-412 --role qa
ship run list --status failed --limit 20 --json
ship run list --status failed --limit 20 --cursor eyJhIjoxfQ
```

When more runs match, the last line gives the `--cursor` for the next page. Give the same filters with it. A status is `running`, `completed`, `stopped`, `failed` or `stale`. A `stopped` run ended on purpose before it finished,
for example because a newer run replaced it. A `stale` run closed without a report, after its time limit. A role is
`planner`, `builder`, `reviewer` or `qa`.

## Read one run

```bash
ship run get builder-ship-412-3f9c41aa-coding-1
```

The command prints the mission and the role of the run, its status, and the reason when a run ended early. It also
prints the harness and the size of the sandbox, the start and end time, and the latest CPU, memory and disk use. The
`session` line is the session of this run, with its outcome and its cost. The `other` line names the sessions of other
runs with the same role. Their cost is not part of the cost of this run.

## Watch a run live

```bash
ship run watch builder-ship-412-3f9c41aa-coding-1
ship run watch qa-ship-412-77d0e2c1-qa-1 --max-seconds 300
ship run watch qa-ship-412-77d0e2c1-qa-1 --since 118
```

The command prints the last events of the agent. Then it prints each new event on one line, for example a thought, a
tool call or an error. It also prints a line when the mission moves to another stage.

When the run ends, the command exits 0 if the run completed, and 7 if it ended in another way. With `--max-seconds`,
the command stops after that time and exits 5 if the run is still active. Its message gives the `--since` value, so
that the next command prints only the events that came after. Use this command to wait for a run, not a loop of
`ship run get` calls.

An agent that uses the SHIP MCP server calls the `watch_agent_run` tool with the id of the run. The call returns when
the run ends, when its mission moves to another stage, or after `maxSeconds` (300 by default, 600 at most). While the
call waits, the server sends each event as a progress notification, if the request has a progress token. The result
has the status of the run, the stage, `ended`, the reason for the return, the last 20 events and a `cursor`. If the run
is still active, call the tool again with `since` set to the cursor.

## Read what the agent did

```bash
ship run logs builder-ship-412-3f9c41aa-coding-1
ship run logs builder-ship-412-3f9c41aa-coding-1 --part system
ship run logs qa-ship-412-77d0e2c1-qa-1 --part prompt
ship run logs builder-ship-412-3f9c41aa-coding-1 --session s-91c2
```

A log has three parts:

| Part | What it has |
| --- | --- |
| `session` (the default) | The event stream of the agent, one JSON event on each line: its thoughts, its tool calls and their results. |
| `system` | The output of the sandbox: the clone, the dependency install, the checks before a push, and git. |
| `prompt` | The text that the repository added to the prompt of the agent, for example from `.ship/agents/<role>.md`. |

The command reads the session of the run itself. To read a session of another run of the same role, name it with
`--session`. The `session` part is long, so filter it before you read it. For example, pipe `--json` output to `jq`, or
search for the error text.

The platform saves the log when the agent ends. While a run is still `running`, the command exits 4 and says so. Read
`ship run get` for its status, and read the log after the run ends. Some old runs did not record their own session. For
such a run, the command also exits 4 and lists the sessions of the role, so that you can name one with `--session`.

## Stop one run

Stop a run only when the user asks for it, because the work in progress of the agent is lost.

```bash
ship run stop builder-ship-412-3f9c41aa-coding-1
```

The status of the run becomes `stopped` a few seconds later. The command stops one run, not the mission. The mission
continues, and it can start a new run for the same stage. To stop the whole mission, use `ship mission stop` from the
`ship-missions` skill.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have the scope of the command, or it is not an admin's token for `ship run stop`. | Ask the user for a token with the scope. Do not retry. |
| 4 | SHIP found no such run, or the run has no saved log yet. | Check the id with `ship run list`. For a log, wait until the run ends. |
| 5 | `ship run watch` stopped at `--max-seconds`, and the run is still active. | Run the command again with the `--since` value from its message. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
| 7 | The run that `ship run watch` followed ended, but it did not complete. | Read the reason with `ship run get`, then its log. |
