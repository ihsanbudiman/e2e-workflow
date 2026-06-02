# e2e-workflow (Claude Code plugin)

[![Repository](https://img.shields.io/badge/GitHub-ihsanbudiman%2Fe2e--workflow-blue)](https://github.com/ihsanbudiman/e2e-workflow)

End-to-end implementation workflow for Claude Code: **explore → clarify → plan → execute → test → review**, with human approval gates and five specialized subagents.

## Install

### From GitHub (recommended)

In Claude Code:

```text
/plugin marketplace add ihsanbudiman/e2e-workflow
/plugin install e2e-workflow@e2e-workflow
```

Or from any terminal:

```bash
claude plugin marketplace add ihsanbudiman/e2e-workflow
claude plugin install e2e-workflow@e2e-workflow
```

Pin to a release tag when available:

```bash
claude plugin marketplace add ihsanbudiman/e2e-workflow@v1.0.0
```

Enable the plugin in Claude Code settings if prompted.

### Local development

Clone the repo and add the marketplace from the checkout path:

```bash
git clone https://github.com/ihsanbudiman/e2e-workflow.git
cd e2e-workflow
claude plugin marketplace add .
claude plugin install e2e-workflow@e2e-workflow
```

## Use

```bash
/e2e Add rate limiting to the API
```

Or start Claude Code and run `/e2e` with no args — it will ask what you want built.

## Workflow

| Phase | Agent                 | Role                                                     |
| ----- | --------------------- | -------------------------------------------------------- |
| 1     | Orchestrator (`/e2e`) | Clarify goal, confirm approach                           |
| 2     | `e2e-explorer`        | Read-only codebase map                                   |
| 3     | `e2e-planner`         | Step-by-step plan (user must say **approved**)           |
| 4     | `e2e-executor`        | Implementation                                           |
| 5     | `e2e-tester`          | Tests + run results                                      |
| 6     | `e2e-reviewer`        | GREEN/RED gate; loops with executor + tester until GREEN |

## Layout

```
e2e-workflow/
├── .claude-plugin/
│   ├── plugin.json       # Plugin manifest
│   └── marketplace.json  # Marketplace (source: ./)
├── commands/
│   └── e2e.md            # /e2e slash command
├── agents/
│   ├── e2e-explorer.md
│   ├── e2e-planner.md
│   ├── e2e-executor.md
│   ├── e2e-tester.md
│   └── e2e-reviewer.md
└── README.md
```

## Uninstall

```bash
claude plugin uninstall e2e-workflow@e2e-workflow
```
