---
name: ship-scheduled-tasks
description: Use when the user wants to schedule a task that SHIP runs every day, week or month, such as a dependency update. It runs SHIP missions on a schedule. Create a scheduled task from a prompt or a template, change its schedule or prompt, or turn it off and on. Run it now, read what its past runs did and cost, and delete it. For example, the user says "every Monday, have SHIP update our dependencies" or "run a flaky test sweep each week". Also use it when the user asks what the last scheduled run did, or which scheduled tasks a project has.
---

# Run missions on a schedule

A scheduled task is a prompt with a cadence. At each run, SHIP files an issue with the prompt as its description, and a
normal mission starts on it. The board, the review, the tests and the proof of that mission are the same as for any
other mission.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship schedule list`, `ship schedule get`, `ship schedule runs`, `ship schedule templates` | `scheduled-tasks:read` |
| `ship schedule create`, `ship schedule update`, `ship schedule delete` | `scheduled-tasks:write` |
| `ship schedule run` | `missions:run` |

Changes also need the admin or owner role in the organization. Each run uses credits, like any mission.

## See the scheduled tasks

```bash
ship schedule list
ship schedule list --project prj_123
ship schedule get aut_123
ship schedule templates
```

`ship schedule list` shows each task with its cadence and its next run. With `--project`, it lists the tasks of one
project. `ship schedule get` also prints the prompt.
`ship schedule templates` shows prepared prompts, each with a suggested cadence.

## Create a scheduled task

```bash
ship schedule create --name deps --template dependency-upgrades
ship schedule create --name triage --file triage.md --every weekly --day monday --time 09:00 --timezone Europe/Amsterdam
ship schedule create --name report --file report.md --every monthly --day 1 --time 06:00 --timezone UTC
```

The schedule is `--every daily`, `weekly` or `monthly`, with `--time` and `--timezone`. A weekly task also needs `--day`
with a weekday. A monthly task needs `--day` with a day from 1 to 28. With `--template`, the task takes the prompt of the
template, and its suggested schedule unless you give one. With `--file` too, the prompt of the file replaces the prompt
of the template. Give `--project <id>`, or set `SHIP_PROJECT_ID`.

Write the prompt like a good issue:

- Write the acceptance criteria as checkboxes, because the QA agent tests against them.
- Say what an empty result is, for example "if no dependency has an update, change nothing".
- Do not ask the agent to change the CI configuration. Nobody watches a scheduled run, and SHIP blocks that change
  without a person.

While the mission of the last run is still open, SHIP skips the next run. Thus a weekly task cannot open a second pull
request that conflicts with the first.

## Change, stop or delete a scheduled task

```bash
ship schedule update aut_123 --file triage.md
ship schedule update aut_123 --every daily --time 07:30 --timezone UTC
ship schedule update aut_123 --disable
ship schedule update aut_123 --enable
ship schedule delete aut_123
```

Only the values that you give change. To change the schedule, give `--every`, `--time` and `--timezone` together, and
`--day` when the interval needs it. `--disable` stops the runs and keeps the task and its history. Delete a task only
when the user asks for it, because its run history goes with it.

## Run a task now, and read its runs

```bash
ship schedule run aut_123
ship schedule runs aut_123
ship schedule runs aut_123 --limit 20 --cursor eyJhIjoxfQ
```

The command `ship schedule run` starts a mission now, outside the schedule. It also runs when the mission of the last
run is still open. The command `ship schedule runs` lists the past runs, each with its status, its mission and the cost
of that mission. When older runs exist, the last line gives the `--cursor` for the next page. A `dispatched` run filed
its issue, but that does not say whether the work passed. Read the mission
with `ship status` for that.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong, or the API refused the schedule. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have the scope of the command, or the role cannot change it. | Ask the user for a token with the scope, from an admin. Do not retry. |
| 4 | SHIP found no such scheduled task, template or project. | Check the id with `ship schedule list`, or give `--project <id>`. |
| 5 | The organization has no credits for a run now. | Tell the user. Check the balance with `ship credits`. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
| 8 | Too many requests, or the organization reached a daily limit. | Wait, then run the command again. Tell the user about a daily limit. |
