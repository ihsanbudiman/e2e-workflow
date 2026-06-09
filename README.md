# e2e-workflow

A Claude Code plugin that runs the **Product Engineering Development Workflow** — a single,
repeatable loop that takes any development work, from a vague idea to a nasty bug, all the way to
a reviewed change that's **ready for you to ship**. One command drives seven phases across seven
specialized subagents, each in its own context, with human approval gates.

The full methodology it implements is in [`COOKBOOK.md`](./COOKBOOK.md).

## Features

### Seven-phase workflow
One command (`/e2e:workflow`) drives the whole thing. An orchestrator owns the conversation and
hands each phase to a focused subagent, carrying results forward. Phases 1–2 are the **product
track** (*is this worth doing, and for whom?*); phases 3–7 are the **engineering track** (*what's
the simplest correct way to build it?*).

| # | Phase | Owner | What it does |
| - | ----- | ----- | ------------ |
| 1 | Understand | Orchestrator + `e2e-explorer` | Find the real problem; map code, conventions, evidence/reproduction |
| 2 | Define | Orchestrator + you | Success criteria, scope (in/out), constraints — the contract for "done" |
| 3 | Design | `e2e-designer` | 2–3 approaches, trade-offs, and a decision record for the chosen one |
| 4 | Decompose | `e2e-planner` | Break the design into small, vertically-sliced, independently testable steps |
| 5 | Build | `e2e-executor` | Implement the approved plan slice by slice |
| 6 | Verify | `e2e-tester` + `e2e-reviewer` | Real tests, then an independent GREEN/RED verdict against the original request |
| 7 | Finalize | `e2e-finalizer` | Confirm deployable state; write hand-off notes (deploy, rollback, what to watch) |

### Human approval gates
You stay in control at the points that matter. Nothing is designed until you confirm the problem,
success criteria, and scope (Gate 1); nothing is decomposed until you approve the approach
(Gate 2); and nothing is built until you explicitly approve the plan (Gate 3). Small, clear tasks
collapse the early gates and pass through in minutes.

### Specialized subagents, least privilege
Each subagent runs in an isolated context with only the tools it needs. The explorer and reviewer
are pure read-only; the designer, planner, and finalizer are read-only on your repo and each write
exactly one session artifact; only the executor and tester change repo files. Heavier-reasoning
roles (designer, planner, reviewer) run on a stronger model; execution roles run faster and cheaper.

### Durable artifact handoff
The decision record (`DESIGN.md`), the approved plan (`PLAN.md`), and the hand-off notes
(`HANDOFF.md`) are written to a per-run session directory, and every downstream agent reads them
from disk — so they stay intact across phases instead of degrading through summaries. A prose
fallback is passed alongside in case a file is unavailable.

### Adversarial review with a loop cap
The reviewer actively tries to prove the work *isn't* done: re-running the suite itself, hunting
edge cases, and checking against the original request rather than just the plan. If it stays RED,
the fix loop caps at three attempts and hands the decision back to you instead of spinning.

### Your git, your call
The workflow never commits for you. Phase 7 confirms the change is deployable and writes the
hand-off notes, but staging, committing, and shipping are left to you. Changes are in your working
tree so you decide what to do with them.

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
