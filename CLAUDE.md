# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **Claude Code plugin** (`e2e`) that turns one slash command into the team's product-engineering
cookbook: **understand → define → design → decompose → build → verify → finalize**, with human
approval gates. There is **no application code, build step, test suite, or linter** — the entire
product is markdown agent/command definitions plus JSON plugin metadata. "Editing the codebase"
means editing prompts and config, and the main risk is breaking the cross-file invariants below,
not breaking a compiler.

The methodology the plugin implements is documented in `COOKBOOK.md`; the phases, gates, and agent
prompts must stay consistent with it.

## Layout

- `COOKBOOK.md` — the canonical methodology (seven phases, templates, the Universal
  Problem-Solving Method, anti-patterns). The plugin implements this; keep them in sync.
- `commands/workflow.md` — the **Orchestrator** prompt. Invoked as `/e2e:workflow <request>`.
  Owns the user conversation, the seven-phase flow, the gates, and is the *only* actor that talks
  to the user or spawns subagents.
- `agents/e2e-{explorer,designer,planner,executor,tester,reviewer,finalizer}.md` — the seven
  subagents. Each runs in an isolated context, cannot talk to the user, and cannot spawn other
  subagents.
- `.claude-plugin/plugin.json` — plugin manifest (`name: "e2e"`, version).
- `.claude-plugin/marketplace.json` — marketplace listing that points at this repo (`source: "./"`).

## Architecture: orchestrator + seven least-privilege subagents

The orchestrator delegates each phase via the Agent tool and carries a *summary* forward — never
the raw subagent context. The design intent of each subagent's frontmatter is deliberate:

| Subagent | Phase | `tools` (least privilege) | `model` / `effort` | Role |
|----------|-------|---------------------------|--------------------|------|
| explorer | 1 Understand | Read, Grep, Glob, Bash (inspection only) | sonnet / medium | read-only context + evidence map |
| designer | 3 Design | Read, Write, Grep, Glob, Bash | opus / high | writes the decision record, else read-only |
| planner  | 4 Decompose | Read, Write, Grep, Glob, Bash | opus / high | writes the plan file, else read-only |
| executor | 5 Build | Read, Write, Edit, Bash, Grep, Glob | sonnet / medium | implements plan + applies fixes |
| tester   | 6 Verify | Read, Write, Edit, Bash, Grep, Glob | sonnet / medium | writes/runs tests |
| reviewer | 6 Verify | Read, Grep, Glob, Bash | opus / high | adversarial GREEN/RED verdict |
| finalizer| 7 Finalize | Read, Write, Grep, Glob, Bash | sonnet / medium | writes hand-off notes, else read-only; never commits |

Phase 2 (Define) is owned by the orchestrator and the user — it has no subagent.

Invariants to preserve when editing these roles:
- **Explorer and reviewer are pure read-only.** Do not add Write/Edit to them — their independence
  is the point (the reviewer is the unbiased gate; the explorer must not mutate state it reports on).
- **Designer, planner, and finalizer are read-only on the repo and each write exactly one session
  artifact** (`DESIGN.md`, `PLAN.md`, `HANDOFF.md` respectively, all under /tmp). Their prompts
  forbid editing any other file. Only the executor and tester change repo files.
- **The finalizer never commits, merges, or pushes** — Phase 7 produces hand-off notes and
  confirms deployable state, but leaves git to the user. This mirrors the workflow's "never
  commits" rule (see below).
- **Heavy-reasoning roles (designer, planner, reviewer, orchestrator) run on opus/high; execution
  roles (explorer, executor, tester, finalizer) run on sonnet/medium.** This cost/quality split is
  intentional.

## The artifact handoff (the load-bearing mechanism)

To avoid degradation through summarization, the load-bearing artifacts are passed **by file path,
not by prose**:

1. Orchestrator creates a per-run dir once at the start of Phase 3:
   `mktemp -d /tmp/e2e-workflow.XXXXXX`. Three files live there:
   - `DESIGN.md` — the decision record, written by the **designer** (approved at Gate 2).
   - `PLAN.md` — the decomposed plan, written by the **planner** (approved at Gate 3). This is the
     authoritative artifact downstream agents implement against.
   - `HANDOFF.md` — the hand-off notes, written by the **finalizer** at the end.
2. Each writer **writes the full file to its path** (overwriting on every revision — never
   appending) and also returns it inline for user review.
3. The planner reads `DESIGN.md` from disk; the executor reads `PLAN.md` and skims `DESIGN.md` for
   the "why"; the tester and reviewer read `PLAN.md` (verifying against it and the ORIGINAL
   request); the finalizer reads both `PLAN.md` and `DESIGN.md`. A short prose summary is passed
   alongside only as a fallback if a file is unavailable.

If you change this convention, change it in **all the places at once**: `commands/workflow.md` and
every agent prompt that reads or writes one of these files. They must agree on the path scheme and
the "read from disk first, prose as fallback" rule.

## Gates and loops (don't quietly remove these)

- **Gate 1 (Phase 2 — Define):** no design begins until the user confirms the problem, success
  criteria, scope, and roughly-known approach (the Definition of Ready).
- **Gate 2 (Phase 3 — Design):** no decomposition until the user approves the chosen approach.
  Trivial work may fold this into Gate 1.
- **Gate 3 (Phase 4 — Decompose):** nothing executes until the user explicitly approves the plan.
  Because the planner overwrites the same file each revision, the file on disk *is* the approved
  plan once this gate passes.
- **Verify→fix loop (Phase 6):** caps at **3 consecutive RED verdicts**, then hands the decision
  back to the user instead of spinning. If the reviewer finds the *plan* is flawed, control returns
  to Phase 4 (Decompose); if the *approach* is fundamentally wrong, to Phase 3 (Design).
- The workflow **never commits** — changes are left in the working tree for the user; Phase 7 only
  writes hand-off notes.

## Naming invariants across files (easy to break)

- The plugin name is **`e2e`** (`plugin.json`). Commands are therefore namespaced `/e2e:workflow`
  and subagents are referenced as **`e2e:e2e-<role>`** (plugin name + agent `name:` frontmatter).
  The orchestrator's delegation targets, each agent file's `name:` field, and the `e2e` prefix
  must all stay in sync — a mismatch silently breaks delegation. The seven roles are
  `explorer, designer, planner, executor, tester, reviewer, finalizer`.
- The **version** appears in both `plugin.json` and `marketplace.json`; bump them together.

## Validating changes

No test harness exists. To sanity-check edits:
- Validate JSON: e.g. `python3 -m json.tool .claude-plugin/plugin.json` (and `marketplace.json`).
- Confirm every `e2e:e2e-<role>` referenced in `commands/workflow.md` has a matching `agents/*.md`
  file whose `name:` matches `<role>`, and vice versa.
- Confirm the seven phases, three gates, and the three session artifacts in `commands/workflow.md`
  stay consistent with `COOKBOOK.md` and the agent prompts.
- To exercise the plugin end-to-end, install it from this local repo as a marketplace in Claude
  Code and run `/e2e:workflow <some request>`; there is no automated alternative.
