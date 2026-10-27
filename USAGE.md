# Marketplace usage

This guide explains how to maintain, test and combine the plugins. Each plugin lives in its own repo, `kamoca-labs/<plugin>`. The paths below start at the root of that repo. The installation steps are in the [README](README.md#installation). The use cases of each command are in the USAGE file of the plugin.

## Contents

- [Update after editing](#update-after-editing)
- [Test without installing](#test-without-installing)
- [Run the hook tests](#run-the-hook-tests)
- [Create a new rule](#create-a-new-rule)
- [Turn a real case into a test](#turn-a-real-case-into-a-test)
- [Combine commands](#combine-commands)
- [Daily workflow](#daily-workflow)
- [Troubleshooting](#troubleshooting)
- [Use cases for each plugin](#use-cases-for-each-plugin)

## Update after editing

Claude Code copies each plugin to `~/.claude/plugins/cache/` **for each version**. Editing the repo changes nothing until you update:

1. Raise the `version` in `.claude-plugin/plugin.json` of the plugin repo (example: `0.1.0` to `0.2.0`).
2. Run `claude plugin marketplace update kamoca`.
3. Run `claude plugin update <name>@kamoca`.
4. Open a new session.

## Test without installing

To try a change before you raise the version:

```bash
claude --plugin-dir ~/Developer/kamoca-labs/git
```

## Run the hook tests

All tests use `node:test`, with no dependencies:

```bash
node --test 'hooks/tests/*.test.ts'                       # all
node --test hooks/tests/commit-body.test.ts              # one rule
node --test hooks/tests/cases.test.ts                    # use cases
```

The [README](README.md#how-the-rules-are-enforced) describes the three levels (rule, use case, structure). The structure test fails if a rule has no hook in `hooks.json` or no `<rule>.test.ts`.

## Create a new rule

1. **Write the test first** in `hooks/tests/<rule>.test.ts`. Use the `helpers.ts` of the folder. Confirm that the test fails.
2. **Write the rule** in `hooks/rules/<rule>.ts`. It reads the payload with `readInput()`, denies with `deny('<rule>', 'reason')`, and ends with no action when it does not apply.
3. **Add the hook** to `hooks/hooks.json`: one entry of its own, with `if:` (Bash) or `matcher` (MCP) and a `statusMessage` `<plugin>:<rule>: ...`.
4. **Add the use cases** to `hooks/tests/cases.json`: at least one scenario that the rule denies and one that it allows. Adjust the `deny` of the existing cases that the new rule affects.
5. **Update the documentation**: one line in the rules table of the plugin README, and the denial case in the USAGE file.
6. Raise the `version` and [update](#update-after-editing).

Skeleton of a rule:

```ts
import { commitMessage, deny, readInput } from '../lib.ts'

const message = commitMessage(readInput())

if (message && /wip/i.test(message.subject)) {
  deny('no-wip', 'A WIP commit does not enter the history.')
}
```

## Turn a real case into a test

Each denial that the hooks make in real use becomes a line in `~/.claude/hook-cases.jsonl`, already in the `cases.json` format:

```json
{"plugin":"git","description":"Title outside the semantic format ...","origin":"real","branch":"feat/x","prompt":"/git:commit","tool_name":"Bash","tool_input":{"command":"..."},"deny":["semantic-title"]}
```

See the last denials:

```bash
tail -n 5 ~/.claude/hook-cases.jsonl | node -e 'require("readline").createInterface({input:process.stdin}).on("line",l=>{const c=JSON.parse(l);console.log(c.plugin, c.deny, c.description)})'
```

| Situation | What to do |
|---|---|
| **Correct denial** | Do nothing. If the scenario is new, save it as a regression test |
| **False positive** (denied what should pass) | Copy the line to `hooks/tests/cases.json`, change `deny` to `[]`, set `origin` to `real <date>`, and fix the rule until the case passes |
| **False negative** (allowed what should be denied) | No line is captured. Write the case by hand in `cases.json` with the expected `deny`, and fix the rule |

The `plugin` field only shows which `cases.json` receives the line. The test ignores it. The `craft` plugin injects context instead of a denial. For `craft`, the line has `inject` in place of `deny`, and the workflow is the same. The first real case in the repo is the false positive of `$VAR` (`origin: "real 2026-10-06"` in the `cases.json` of `git`).

## Combine commands

Only the first `/command` of the line runs. The other commands become arguments, and the skill that runs passes each one to the matching command.

| You type | What happens |
|---|---|
| `/git:commit` | Commits and pushes on the current branch (creates a branch if you are on main) |
| `/git:commit /<plugin>:<command> @reviewer` | Commits, pushes, and runs the command of the other plugin with the remaining arguments |
| `/git:commit /git:main` | Commits directly on main, with a justification and an audit |
| `/git:split /<plugin>:<command>` | Splits the branch and runs the command for each part, in order |

## Daily workflow

```
/craft:atdd reject attachment above 10 MB      specification, red test, green and refactor
/git:commit                                    creates the branch, commits and pushes
... review and merge ...
/git:sync                                      goes back to the updated main and deletes the branch
```

## Troubleshooting

| Symptom | Cause | Solution |
|---|---|---|
| `/git:commit` does not appear | The plugin was installed in the current session | Open a new session |
| I changed the repo and nothing changed | The plugin runs from the cached copy | [Update the version](#update-after-editing) |
| `[git:<rule>] ...` denied the commit | The message violates a rule | See the [denied messages](https://github.com/kamoca-labs/git/blob/main/USAGE.md#when-the-hook-denies) |
| A hook denied something that should pass | False positive of the rule | [Turn the case into a test](#turn-a-real-case-into-a-test) and fix it |
| `Marketplace name impersonates...` | `claude-plugins` is a reserved name | The marketplace name is `kamoca`. The repo is `kamoca-labs/marketplace` |
| Commit denied by another hook (`[commit-hook]`) | A hook of the open project denied it, not a hook of this repo | Run the command yourself with `!` in the prompt |

## Use cases for each plugin

- [git/USAGE.md](https://github.com/kamoca-labs/git/blob/main/USAGE.md)
- [craft/USAGE.md](https://github.com/kamoca-labs/craft/blob/main/USAGE.md)
- [docker/USAGE.md](https://github.com/kamoca-labs/docker/blob/main/USAGE.md)
