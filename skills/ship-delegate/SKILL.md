---
name: ship-delegate
description: Hand work to the SHIP AI agent team from this repository. Delegate a plan or a spec that you wrote, so SHIP builds it. Delegate a branch with an open pull request, so SHIP reviews it, tests it and makes it ready to merge. Use when the user says "delegate this", "hand this to SHIP", "let SHIP build this", "let SHIP review this" or "ship this spec". Also use it when the user finishes a plan or a change and asks what to do next.
---

# Hand work to SHIP

Two hand-offs, each one command. SHIP starts a mission for the work, and its agents plan, build, review and test it.

This skill needs the `ship` CLI and a `SHIP_API_TOKEN`. If you do not have them, do the setup at
<https://letsship.ai/SKILL.md> first. If this file has no `version` in its frontmatter, you read the hosted copy, so do
the setup too. If its `version` is older than `ship --version`, run `ship init --force`.

| Command | Token scope |
| --- | --- |
| `ship delegate plan` | `missions:run` |
| `ship delegate pr` | `missions:run` |

## When to use it

- The user wrote a plan or a spec and wants SHIP to build it, not you.
- The user finished code on a branch and wants SHIP to review it, test it and make it ready to merge.

Do not delegate work that you are in the middle of, unless the user asks. Finish or stop first, so that the hand-off
has a complete plan or a complete branch. To read the state of a mission or to send it feedback after the hand-off,
use the `ship-missions` skill.

## Hand over a plan

Write the plan to a file first, then delegate it. SHIP skips its own Planner and builds from your spec as you wrote it.

```bash
# For an issue that already exists
ship delegate plan SHIP-412 --file docs/plan.md

# No issue yet: create one and start the mission in the same call
ship delegate plan --title "Rate-limit the search endpoint" --file docs/plan.md
```

If you know the files that the plan changes, name them with `--file-plan <path.json>`. The file is a JSON array of
`{path, action, description}`. Without it, the Builder chooses the files.

## Hand over a branch

Open the pull request yourself, then delegate it. SHIP never pushes a branch or opens a pull request for you. If the
`gh` CLI is available, the command reads `--pr` from the current branch.

```bash
ship delegate pr SHIP-412              # review, then test, then merge
ship delegate pr SHIP-412 --at qa      # already reviewed: only test it
ship delegate pr                       # no issue: SHIP names the mission after the pull request
```

If CI fails on that pull request, SHIP fixes the failure before the review. That is expected, not an error.

## Choose the harness, model or sandbox for one mission

These flags change how this one mission runs. They take priority over the `ship.yml` of the project and do not change
the file. They apply to the role that the mission starts at: the Builder for `delegate plan`, and the Reviewer or the
QA agent for `delegate pr`.

```bash
ship delegate plan SHIP-412 --file docs/plan.md --harness codex
ship delegate plan SHIP-412 --file docs/plan.md --model opus
ship delegate pr SHIP-412 --at qa --harness open-code --model z-ai/glm-5.2
ship delegate plan SHIP-412 --file docs/plan.md --sandbox python-uv --size standard-2
```

The harnesses are `claude-code`, `codex`, `pi`, `kilo-code` and `open-code`. You can give `--model` without
`--harness`, and the configured harness stays. `--sandbox` names a custom sandbox (see the `ship-sandboxes` skill), and
`--size` sets the machine size for this mission.

Use these flags when the user asks for a specific agent, model or sandbox, or to compare two of them on the same work.
Otherwise, leave them out. The configuration of the project is the setting that the team agreed on.

## The configuration of the project

That configuration is `ship.yml` at the root of the repository. It sets the harness and the model of each role. It
also sets how long the retry loop continues, where a preview environment deploys, and how QA tests a change that has no
preview URL.

The published JSON Schema is the source of truth for its shape:

```
https://letsship.ai/schema/ship.yml.json
```

When you write or change the file, put the modeline on the first line, so that an editor validates the file. Then
validate the file against the schema before you hand anything over:

```yaml
# yaml-language-server: $schema=https://letsship.ai/schema/ship.yml.json
version: 1
deployments: {}
```

`version` and `deployments` are both required. A project that has nothing to preview writes `deployments: {}` and keeps
the key. If the file fails validation, SHIP ignores all of it, and `agents` and `retries` go back to the platform
defaults. The full field reference is at <https://letsship.ai/docs/configuration/schema>.

A change to `ship.yml` is normal work in the repository: a branch and a pull request, or a mission that you delegate.
The flags in this skill never write to the file.

## Give one role extra instructions for one mission

When the user wants this mission built or judged in a different way, send the instructions with the hand-off. Examples
are "review it for security" or "keep the public API stable". Do not put them in the issue text. They apply to the role
that the mission starts at, for this mission only, and SHIP commits nothing to the repository.

```bash
ship delegate pr SHIP-412 --prompt-file security-review.md
ship delegate pr SHIP-412 --prompt-path .ship/lanes/security.md
```

Write the instructions to a file first, the same way as a plan. `--prompt-path` names a file that is already in the
repository. Add `--prompt-ref` to read that file at a specific git ref.

`--prompt-mode` decides what happens to the own instructions of the repository for that role, in
`.ship/agents/<role>.md`:

| Mode | Effect |
| --- | --- |
| `replace` (default) | Your instructions are the only project instructions for the role. SHIP does not read the file of the repository. |
| `extend` | SHIP reads the file of the repository first and adds your instructions after it. |

```bash
ship delegate pr SHIP-412 --prompt-file security-review.md --prompt-mode extend
```

Use `extend` when you add a concern, such as a security review or one more rule. The `replace` mode drops the
conventions that the repository wrote for the role, and the user almost never wants that. Use `replace` only for a
mission that is a different job from the normal work, or when the user tells you to ignore the file. The SHIP charter
for the role applies in both modes.

## Exit codes

Every command also takes `--json`, which prints the raw API response.

| Exit code | Meaning | What to do |
| --- | --- | --- |
| 2 | The arguments are wrong. The message names the problem. | Fix the command. |
| 3 | The token is not set, or it does not have `missions:run`. | Ask the user for a token with the scope. Do not retry. |
| 4 | SHIP found no such issue or project. | Check the issue id, or give `--project <id>`. |
| 5 | The hand-off needs something that is not true yet, for example a review with no pull request. | Do what the message says, then run the command again. |
| 6 | The organization did not finish the onboarding. | Ask the user to finish it in the console. |
