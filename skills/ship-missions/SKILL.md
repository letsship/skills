---
name: ship-missions
description: Use when the user wants to follow a SHIP mission, check its status, or pass it feedback, review comments or a decision while it runs. It follows, controls and finishes the SHIP missions that you or the user started. List missions, read the state of one, and send it instructions or feedback. Pause, resume, stop, restart, merge or decline a mission, approve a checkpoint or a credit limit, and read its preview, proof and credits. For example, the user asks "what is the status of SHIP-412", "is SHIP done with it" or "tell SHIP to change X". Also use it when the user says "pause that mission" or "merge it", or asks for a change in one line.
---

# Follow and steer a mission

A mission is the work that SHIP does for one issue. Use this skill after a hand-off (see the `ship-delegate` skill), to
read the state of a mission, to send it new information and to finish it. To read what one agent did inside a mission,
use the `ship-agent-runs` skill.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship status`, `ship mission list` | `missions:read` |
| `ship instruct`, `ship mission pause`, `ship mission resume`, `ship mission stop` | `missions:write` |
| `ship mission merge`, `ship mission decline`, `ship mission auto-merge` | `missions:write` |
| `ship mission archive`, `ship mission unarchive`, `ship mission disposition` | `missions:write` |
| `ship mission create`, `ship mission start`, `ship mission approve-ceiling` | `missions:run` |
| `ship mission override get` | `agents:read` |
| `ship mission override set` | `agents:read` and `agents:write` |
| `ship mission override clear` | `agents:write` |
| `ship mission inspect` | `inspect:read` |
| `ship preview list`, `ship preview get` | `environments:read` |
| `ship preview destroy` | `environments:write` |
| `ship artifact list`, `ship artifact get` | `artifacts:read` |
| `ship credits`, `ship credits offers`, `ship credits request`, `ship setup`, `ship notice list`, `ship notice dismiss` | any token |
| `ship credits ledger`, `ship credits buy`, `ship credits plan` | any token, with an owner or admin behind it |

The controls also need a role in the organization that can control missions. Archive and disposition need an owner or
an admin. Without `--project`, the override commands read the project from the mission, so the token also needs
`missions:read`. A token never has more access than the person who created it.

## See the missions

```bash
ship mission list
ship mission list --activity needs_attention
ship mission list --search "rate limit" --json
ship mission list --project prj_123 --offset 25
```

The list shows each mission with what it does now, its stage and its pull request. The `--activity` flag takes the
values of the status filter in the console, for example `building`, `reviewing`, `testing`, `ready_to_merge`,
`needs_attention` or `paused`. The list covers every project of the token, unless `--project` names one.
`SHIP_PROJECT_ID` does not narrow a list. When more missions match, the last line gives the `--offset` of the next
page.

## Read the state of a mission

```bash
ship status SHIP-412
ship status SHIP-412 --json    # for jq
```

The command prints the stage of the mission, its review state, its test state and its pull request. It also lists the
agent runs of the mission, and the run whose artifacts are the proof of the work. Read it before you tell the user that
work is complete.

## Start a mission from a request

When the user asks for a change in one line and there is no plan or branch yet, let SHIP plan it:

```bash
ship mission create --title "Fix the login redirect" --description "- [ ] Login returns to the page it came from"
ship mission start SHIP-412 --project prj_123
```

`ship mission create` files the issue and starts the mission at the Planner. Write the acceptance criteria as
checkboxes, because the QA agent tests against them. `ship mission start` starts a mission on an issue that is already
in the tracker. When the user has a plan or a branch, use the `ship-delegate` skill instead.

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

## Pause, resume or stop a mission

```bash
ship mission pause SHIP-412
ship mission resume SHIP-412
ship mission stop SHIP-412
```

A pause lets the current stage finish, then holds the mission before its next stage. A resume continues it. A stop ends
every agent of the mission, and their work in progress is lost. The pull request stays open. Pause or stop a mission
only when the user asks for it.

A checkpoint in `ship.yml` pauses a mission after a stage, so that a person can read the result first. `ship status`
then shows `paused yes (checkpoint:<role>)`. When the user approves the result, resume the mission. When the user wants
changes to the result, send an instruction instead.

## Change the agents of one mission

An override changes the harness, the model or another setting of one role, for one mission only. Each field replaces
the same field of `ship.yml`, and the file itself does not change. The next run of that role uses the override.

```bash
ship mission override get SHIP-412
ship mission override set SHIP-412 --role reviewer --harness codex --model gpt-5.5
ship mission override set SHIP-412 --role builder --checkpoint on
ship mission override set SHIP-412 --role qa --prompt-file qa-notes.md --prompt-mode extend
ship mission override set SHIP-412 --role builder --harness codex --unset model
ship mission override clear SHIP-412
```

The action `set` changes only the role that `--role` names, and only the settings that you give. The other roles and
settings of the override stay. The flag `--checkpoint on` pauses the mission after that stage, so that a person reads
the result first. The flag `--reject-on` sets the lowest finding that makes the reviewer ask for changes. The flag
`--base-branch` sets the branch that the mission starts from. The flag `--unset` removes stored fields, for example the
model when you change the harness, and the action `clear` removes the whole override.

Change an override only when the user asks for it, because a different model can change the cost of the mission.
Without `--project`, the command reads the project from the mission. The variable `SHIP_PROJECT_ID` does not apply to
an override, because an override saved for the wrong project changes no run.

## Auto-merge, archive and disposition

```bash
ship mission auto-merge SHIP-412 on
ship mission auto-merge SHIP-412 off
ship mission archive SHIP-412
ship mission unarchive SHIP-412
ship mission disposition SHIP-412 superseded --superseded-by SHIP-420
```

With auto-merge on, SHIP merges the pull request of the mission when the checks, the review and the test pass, and no
person merges it. For this mission, the setting replaces the auto-merge of the project. Turn it on only when the user
asks for it.

An archived mission is not on the board and not in `ship mission list`. The disposition records why a mission ended,
for the delivery statistics: `product`, `probe`, `superseded`, `takeover`, or `none` to clear it. Record only what
the user decided, never your own guess.

## Restart a mission

A mission that stopped, failed or needs attention does not start again on its own. When the user asks for a new
attempt, start it again:

```bash
ship mission start SHIP-412 --project prj_123
```

Read `ship mission inspect` first if you must find why the last attempt failed. It needs the `inspect:read` scope.

```bash
ship mission inspect SHIP-412 --sections taskState,retryReason
```

## Let a mission continue past its credit limit

Each mission can use a fixed number of credits. At that limit, the mission pauses until a member lets it continue.
`ship status` and `ship notice list` show it. When the user agrees to the extra cost, run:

```bash
ship mission approve-ceiling SHIP-412 --project prj_123
```

The command raises the limit of the mission by one step, and the stage that stopped runs again. Without `--project` or
`SHIP_PROJECT_ID`, the command reads the project from the mission, so the token also needs `missions:read`. Check the balance
with `ship credits` before and after work:

```bash
ship credits
ship notice list
ship notice dismiss credits_low:grant_123
```

The notices show what needs attention in the organization, such as a tracker to connect again or credits that run low.
You can close a warning with `ship notice dismiss`. A critical notice stays until someone fixes its cause.

## Credits

```bash
ship credits ledger
ship credits ledger --limit 100 --cursor eyJhIjoxfQ
ship credits offers
ship credits request -m "Two more missions for the release this week"
```

The ledger shows where the credits went, newest first: each mission and the credits that it used, each grant, each
adjustment and each expiry. When more entries exist, the last line gives the `--cursor` for the next page. The offers
are the credit packs, the monthly plans and Enterprise, with their prices. A credit request goes to SHIP or to the
owners and admins of the organization, and the reply says which.

Buy credits or change the plan only when the user asks you to:

```bash
ship credits buy credit_pack_50 --start-now
ship credits buy team --plan starter --start-now
ship credits plan scale
ship credits plan --cancel
```

The command `ship credits buy` prints the address of a payment page and charges nothing. Give that address to the user,
who pays there. Without `--start-now`, the command shows the text of the consent to start the credits at once, and
stops. Show that text to the user, and add `--start-now` only when the user agrees to it. The command
`ship credits plan` changes the monthly plan from the next renewal. With `--cancel`, the plan ends at the end of the
month that the user paid for.

## Check the preview and the proof

```bash
ship preview list --mission SHIP-412
ship artifact list --mission SHIP-412
ship artifact list --run qa-ship-412-77d0e2c1-qa-1
```

SHIP deploys a preview of the pull request through the delivery pipeline of the repository, and the QA agent tests it.
The proof of a mission is the artifacts of the QA run that passed: traces, screenshots, videos and logs. The command
`ship status` names that run, so give its id to `--run`. Each artifact has a link that opens it. Give these links to the
user when they ask for evidence.

`ship preview destroy` removes a preview before it expires. Use it only when the user asks for it.

## Merge or decline the pull request

```bash
ship mission merge SHIP-412
ship mission decline SHIP-412
```

Merge or decline only when the user tells you to. The merge uses the merge method of the repository, and then the
mission becomes done. If the pull request cannot merge, the command exits 5 with the reason. On a merge conflict, SHIP
also starts the Builder to fix the conflict, and it merges the pull request when the mission is ready again. Do not
merge again, and tell the user that the Builder works on the conflict. The decline closes the pull request without a
merge, and you cannot undo it.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have the scope of the command, or the role cannot do it. | Ask the user for a token with the scope. Do not retry. |
| 4 | SHIP found no such mission, project, preview or artifact. | Check the issue id or the other id, or give `--project <id>`. |
| 5 | The mission is not in a state that allows the action, or the organization has no credits. | Read the message, then `ship status`. Do what the message says. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
| 8 | Too many requests, or the organization reached a daily limit. | Wait, then run the command again. Tell the user about a daily limit. |
