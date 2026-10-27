# kamoca

## Contents

- [Installation](#installation)
- [Context](#context)
- [Catalog](#catalog)
- [How the rules are enforced](#how-the-rules-are-enforced)
- [Plugin structure](#plugin-structure)
- [Documentation](#documentation)

## Installation

```bash
claude plugin marketplace add kamoca-labs/marketplace
claude plugin install git@kamoca
claude plugin install craft@kamoca
claude plugin install docker@kamoca
```

Node 22.18 or later is required. The hooks are `.ts` files that Node runs directly. The commands appear in the next session. The repos are private: Claude Code uses your GitHub credentials (`gh auth login`).

## Context

This is a personal marketplace of [Claude Code](https://code.claude.com) plugins, grouped by domain. Each plugin combines skills (the `/domain:action` commands) and hooks. The hooks **enforce** the rules of that domain, so the model does not need to remember them.

The commit skills started as loose files in `~/.claude`, with no versions. This repo lists the plugins in the `kamoca` marketplace, with one repo for each plugin in the [kamoca-labs](https://github.com/kamoca-labs) organization. This gives you the following:

- Everything is versioned and reviewed in a PR/MR, like any other code.
- Each domain (git, craft, docker) evolves and installs on its own.
- The history rules (semantic commit, body that explains why, nothing directly on main) and the good practices (ATDD, refactor) apply in **all** projects, at work and personal.

Only general content belongs here. Content that belongs to one place only (for example, a work GitLab integration) stays in its own marketplace. You combine it with these plugins through the command arguments.

## Catalog

| Plugin | Command | What it does |
|---|---|---|
| [`git`](https://github.com/kamoca-labs/git#readme) | `/git:commit` | Atomic semantic commits with a body, on a dedicated branch, then push |
| | `/git:mr` | Opens the PR/MR through the MCP or the CLI, with reviewers, draft and target. No squash and no AI mention |
| | `/git:review` | Reviews the PR/MR of another person from the link, and asks for confirmation before it publishes |
| | `/git:ci` | Diagnoses a failed CI and runs the intermittent failures again |
| | `/git:daily` | Writes the daily update from the local commits and the PR/MR on GitHub and GitLab |
| | `/git:release` | Version, changelog, tag and release on the forge |
| | `/git:main` | Allows commit and push on main, with a justification and an audit |
| | `/git:sync` | After the merge, goes back to the updated main and deletes the merged branch |
| | `/git:split` | Splits a large branch into chained branches, one for each subject |
| | `/git:undo` | Undoes a commit, amend, rebase or reset through the reflog, with a backup |
| [`craft`](https://github.com/kamoca-labs/craft#readme) | `/craft:craft` | Root rule (less code is better than more code) and triage of SOLID, KISS, YAGNI, DRY and Clean Code. It loads only the skill of each principle with a signal. The hook runs it automatically when the suite turns green |
| | `/craft:setup` | Prepares a new project: tests, a linter with size and complexity limits, and the `test` and `lint` scripts |
| | `/craft:atdd` | Runs the 7-step ATDD workflow, from the Given/When/Then specification to the final green. It has tasks for each scenario and a goal that holds the end of the turn until the condition is true |
| | `/craft:style` | Style mode for visual-only changes: iterates the screen with impeccable, with no test first, and sends behavior to `/craft:atdd` |
| | `/craft:memory` | Long-term project memory in `.claude/memory/`, inside the repo. The hook loads the catalog in every session and denies memory outside the project |
| | (automatic) | A hook loads the `craft:ste` skill in every session: replies and texts follow ASD-STE100 |
| [`docker`](https://github.com/kamoca-labs/docker#readme) | `/docker:docker` | Single entry point: routes to create or tune, with private sub-skills that both share |
| | `/docker:create` | Newest image versions, nginx with HTTP/2 and HTTP/3. Dev has hot reload and the host UID/GID. Prod is lean, in `./deploy`, with its own `.env`, defaults, volumes in a local folder and standard names |
| | `/docker:tune` | Puts an existing Docker setup under the rules and optimizes it with measurements, one change at a time |

The commands combine in the arguments, also with the commands of other marketplaces:

```
/git:commit /<plugin>:<command> @reviewer
```

## How the rules are enforced

| Layer | Role |
|---|---|
| **Skill** (`skills/*/SKILL.md`) | Tells Claude **how** to work: steps, format, templates |
| **Hook** (`hooks/rules/<rule>.ts`) | **Denies** the call that violates the rule, before it runs |

Each rule is one file, with its own hook in `hooks.json` and its own test in `hooks/tests/<rule>.test.ts`. When a call violates several rules, all of them appear at once. Each one has its own prefix (`[git:semantic-title]`, `[git:commit-body]`).

The hook uses the `if:` field of Claude Code (`Bash(git commit *)`, `Bash(git push *)`) to run only when needed. This filter is best-effort, so each rule checks again that the command is really a commit or a push.

Authorizations such as `/git:main` are read from the **last prompt that you typed** in the session transcript. Claude cannot authorize itself.

Every hook has tests on three levels:

| Level | File | What it guarantees |
|---|---|---|
| Rule | `tests/<rule>.test.ts` | The isolated rule denies and allows what it must |
| Use case | `tests/cases.json` | The real scenario (command, branch, prompt) goes through **all** hooks, and only the expected rules deny |
| Structure | `tests/structure.test.ts` | No rule is left without a dedicated hook or a test |

Each denial in real use is saved to `~/.claude/hook-cases.jsonl`, already in the `cases.json` format. A hook can deny what it should allow, or allow what it should deny. In both cases, the case goes into `cases.json` and becomes a test. See [turn a real case into a test](USAGE.md#turn-a-real-case-into-a-test).

## Plugin structure

This repo has only the `.claude-plugin/marketplace.json` file. It points to one repo for each plugin in `kamoca-labs/<plugin>`. Each plugin has the same structure, at the root of its repo:

```
.claude-plugin/plugin.json
skills/<command>/SKILL.md        one for each command, plus the private sub-skills
hooks/
├── hooks.json                   one hook for each rule
├── lib.ts                       payload reading, message, prompt
├── rules/<rule>.ts              one rule for each file
└── tests/
    ├── <rule>.test.ts           one test for each rule
    ├── cases.json               end-to-end use cases
    ├── cases.test.ts            runs cases.json against hooks.json
    ├── capture.test.ts          a real denial becomes a case
    └── structure.test.ts
```

## Documentation

| Document | Content |
|---|---|
| [USAGE.md](USAGE.md) | Update, test, create a new rule, turn a real case into a test, combine commands, troubleshooting |
| [git/README.md](https://github.com/kamoca-labs/git#readme) | Catalog and rules of the `git` plugin |
| [git/USAGE.md](https://github.com/kamoca-labs/git/blob/main/USAGE.md) | Use cases of `git`, with examples of accepted and denied messages |
| [craft/README.md](https://github.com/kamoca-labs/craft#readme) | Commands, skills for each principle, refactor hook and the supported test runners |
| [craft/USAGE.md](https://github.com/kamoca-labs/craft/blob/main/USAGE.md) | ATDD in a feature and in a bug, automatic and on-demand refactor |
| [docker/README.md](https://github.com/kamoca-labs/docker#readme) | Commands and rules of the `docker` plugin |
| [docker/USAGE.md](https://github.com/kamoca-labs/docker/blob/main/USAGE.md) | Create, start dev and prod, tune with measurements, hook denials |
