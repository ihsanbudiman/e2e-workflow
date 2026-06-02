# e2e-workflow

A Claude Code plugin that runs a full implementation as a guided pipeline:
**explore → clarify → plan → execute → test → review** — with human approval gates
and five specialized subagents, each in its own context.

## Features

### Six-phase workflow
One command (`/e2e:workflow`) drives the whole thing. An orchestrator owns the
conversation and hands each phase to a focused subagent, carrying results forward.

| Phase | Subagent | What it does |
| ----- | -------- | ------------ |
| Clarify | Orchestrator | Restates the goal, surfaces assumptions, proposes approaches |
| Explore | `e2e-explorer` | Read-only map of the code, conventions, and build/run/test commands |
| Plan | `e2e-planner` | Step-by-step plan with success criteria |
| Execute | `e2e-executor` | Implements the approved plan |
| Test | `e2e-tester` | Writes and runs real tests, reports pass/fail |
| Review | `e2e-reviewer` | Independent GREEN/RED verdict against the original request |

### Human approval gates
You stay in control at the points that matter. Nothing gets built until you confirm
the goal and approach, and nothing executes until you explicitly approve the plan.

### Specialized subagents, least privilege
Each subagent runs in an isolated context with only the tools it needs. Explorer,
planner, and reviewer are read-only; only the executor and tester can write. Heavier
reasoning roles (planner, reviewer) run on a stronger model; execution roles run
faster and cheaper.

### Durable plan handoff
The approved plan is written to a per-run session file, and every downstream agent
reads it from disk — so the plan stays intact across phases instead of degrading
through summaries. A prose fallback is passed alongside it in case the file is
unavailable.

### Adversarial review with a loop cap
The reviewer actively tries to prove the work *isn't* done: re-running the suite
itself, hunting edge cases, and checking against the original request rather than just
the plan. If it stays RED, the fix loop caps at three attempts and hands the decision
back to you instead of spinning.

### Your git, your call
The workflow never commits for you. Changes are left in your working tree so you
decide what to stage and commit.

## Install

In Claude Code:

```text
/plugin marketplace add ihsanbudiman/e2e-workflow
/plugin install e2e@e2e-workflow
```

## Use

```text
/e2e:workflow Add rate limiting to the API
```

Or run `/e2e:workflow` with no arguments and it will ask what you want built.

## Uninstall

```text
claude plugin uninstall e2e@e2e-workflow
```

## License

MIT