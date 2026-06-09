# The Product Engineering Development Workflow

> A single, repeatable loop for taking any development work — from a vague idea or a nasty bug — all the way to a reviewed change that's **ready for you to ship.**

This is the methodology the **`e2e` plugin** implements. The `/e2e:workflow` orchestrator
walks a request through these seven phases, delegating each to a focused subagent and stopping
at human gates. The mapping:

| Phase | Owner | Gate |
| ----- | ----- | ---- |
| 1. UNDERSTAND | orchestrator + `e2e-explorer` | — |
| 2. DEFINE | orchestrator + user | Gate 1 — Definition of Ready |
| 3. DESIGN | `e2e-designer` | Gate 2 — design approved |
| 4. DECOMPOSE | `e2e-planner` | Gate 3 — plan approved |
| 5. BUILD | `e2e-executor` | — |
| 6. VERIFY | `e2e-tester` + `e2e-reviewer` | GREEN verdict |
| 7. FINALIZE & HAND-OFF | `e2e-finalizer` | — (never commits) |

**Scope:** This is an *end-to-end development* workflow. It ends the moment the change is
dev-complete and handed off. **Shipping and measuring are yours** — this loop gets the work to
the point where you can confidently take it to production.

**How to use this:** Run every meaningful piece of work through the same loop. Small tasks pass
through quickly (minutes); large ones spend real time in each phase. The goal is the same every
time — **build the right thing, and build it right.**

---

## The Core Loop

```
 1.UNDERSTAND → 2.DEFINE → 3.DESIGN → 4.DECOMPOSE → 5.BUILD → 6.VERIFY → 7.FINALIZE
      (why / what)               (how)                     (correct & ready to hand off)
            ▲                                                          │
            └──────────── loop back when something's wrong ───────────┘
```

Two questions run through the whole loop:

- **Product track —** *Is this worth doing, and for whom?* (phases 1–2)
- **Engineering track —** *What's the simplest correct way to build it?* (phases 3–7)

Development is iterative: when Verify fails or scope shifts, loop back to the right phase rather
than forcing forward. Never skip phase 1 — most wasted effort is a well-built solution to a
poorly understood problem.

---

## Phase 1 — Understand

Get the real problem before touching a solution. Separate the *symptom* from the *cause* and the
*request* from the *need*.

- [ ] State the problem in one sentence, in the user's words.
- [ ] Identify **who** has this problem and how often.
- [ ] Ask "why does this matter?" until you hit a real outcome (time, risk, trust, money).
- [ ] Gather evidence — data, tickets, logs, a reproduction. Don't run on assumptions.
- [ ] List what you *don't* yet know and what you're assuming.

> **Rule of thumb:** if you can't explain the problem without describing your solution, you don't understand it yet.

---

## Phase 2 — Define

Turn the problem into a crisp, bounded target. This is your contract for "done."

- [ ] Write the **success criteria** — what's measurably true when this is solved.
- [ ] Set **scope**: what's in, what's explicitly out (the out-list matters most).
- [ ] Note **constraints**: deadline, tech, dependencies, compliance.
- [ ] Define the **smallest version** that delivers value (your first slice).
- [ ] Quick gut-check with a stakeholder before building.

**Definition of Ready** (don't start building until all true):
problem is clear · success is measurable · scope is bounded · approach is roughly known · dependencies identified.

---

## Phase 3 — Design

Choose *how*. Spend design effort proportional to cost-of-being-wrong. A one-line fix needs no
doc; a new service needs a real one.

- [ ] Sketch **2–3 approaches**; note tradeoffs (effort, risk, performance, maintainability).
- [ ] Pick one and write down **why** (this becomes your decision record).
- [ ] Identify the **riskiest assumption** and de-risk it early (spike, prototype).
- [ ] Plan for failure: errors, edge cases, observability, how the change can be undone.
- [ ] Flag anything that needs review *before* you build it, not after.

> Prefer the **boring, reversible** option. Save your novelty budget for the part that's actually novel.

---

## Phase 4 — Decompose

Break the work into small, independently verifiable steps.

- [ ] Slice **vertically** (thin end-to-end value) over horizontally (all of layer X first).
- [ ] Each piece should be reviewable in well under a day's worth of changes.
- [ ] Order by **risk and dependency** — do the scary/unblocking parts first.
- [ ] Each piece has its own clear "done" and its own test.
- [ ] Keep a visible list/board so progress and blockers are obvious.

---

## Phase 5 — Build

Implement one slice at a time, keeping the system working the whole way.

- [ ] Make the change small and focused — one concern per change.
- [ ] Write the test alongside (or before) the code.
- [ ] Keep it readable: names, structure, and comments explain *why*, not *what*.
- [ ] Commit in logical chunks with clear messages.
- [ ] Update docs/config as you go, not "later."
- [ ] Hit a snag? Run the **Universal Problem-Solving Method** below instead of guessing.

---

## Phase 6 — Verify

Prove it works *and* that you didn't break anything else. Quality is checked here, not hoped for.

- [ ] Tests pass (unit + integration for the touched paths).
- [ ] Manually exercise the happy path **and** the nasty edges.
- [ ] Self-review the diff first — you'll catch half the issues yourself.
- [ ] Open it for **peer review**; address feedback, don't just defend it.
- [ ] Confirm performance, security, and accessibility where relevant.

**Pre-merge checklist:** tests green · diff self-reviewed · scope unchanged from PR description · no debug/secret leftovers · docs updated · reviewer approved.

---

## Phase 7 — Finalize & Hand-off

Get the change to a clean, deployable state and hand it off. **This is where development ends — you take it from here to ship.**

- [ ] Confirm review and checks are green. *(In the `e2e` plugin the merge/commit is left to you — the workflow never commits.)*
- [ ] Update docs, changelog/config, and any migration notes.
- [ ] Confirm it's in a **deployable state** (builds clean, migrations ordered, flags set).
- [ ] Write **hand-off notes** for shipping: what changed, how to deploy, how to roll back, what to watch.
- [ ] Signal it's ready and file any follow-ups you deliberately deferred.

**Dev-complete checklist:** reviewed GREEN · docs/changelog updated · deployable · rollback documented · hand-off notes written · follow-ups logged.

---

## The Universal Problem-Solving Method

This nests inside **any** phase — a stuck build, a flaky test, an ambiguous spec, a mysterious bug. When you're blocked, run this instead of guessing:

1. **Clarify** — What exactly is wrong? What does "fixed" look like? State it precisely.
2. **Reproduce / Observe** — Make the problem happen reliably. Gather evidence. *Never debug by assumption.*
3. **Isolate** — Narrow the surface. Binary-search the cause: bisect commits, disable halves, add probes.
4. **Hypothesize** — Form one testable theory of the cause. Write it down.
5. **Test the smallest change** — Change one thing. Confirm it does what you predicted.
6. **Verify** — Is it actually fixed? Did you cause side effects? Does the original symptom stay gone?
7. **Learn** — Record root cause and fix. Add a test or guard so it can't silently return.

> **The two deadliest traps:** fixing the symptom instead of the cause, and changing several things at once so you can't tell what worked. Resist both.

---

## Templates

### Problem Brief (Phase 1–2)
```
Problem:        [one sentence, user's words]
Who / how often:[affected users + frequency]
Why it matters: [the real outcome at stake]
Success looks like: [measurable criteria]
In scope:       [...]
Out of scope:   [...]
Constraints:    [deadline / tech / risk]
Smallest first slice: [...]
```

### Decision Record (Phase 3)
```
Decision:   [what we chose]
Date / owner: [...]
Context:    [the problem + constraints]
Options considered: [A / B / C + tradeoffs]
Choice & why: [the pick and the reasoning]
Consequences: [what this commits us to, what we give up]
Revisit when: [trigger that would change this]
```

### Change / PR Description (Phase 5–6)
```
What:   [the change in one line]
Why:    [link to problem/brief]
How:    [approach + anything non-obvious]
Scope:  [what this does NOT touch]
Testing:[how it was verified]
Risk / rollback: [blast radius + how to undo]
```

### Hand-off Notes (Phase 7 — what *you* need to ship it)
```
What's ready:   [the change in one line]
Affected:       [users / areas / services]
How to deploy:  [steps / flag to flip / migration order]
Rollback:       [how to undo + how fast]
Watch after:    [signals/dashboards to keep an eye on]
Known limits / follow-ups: [...]
```

---

## Operating Principles

- **Smallest reversible step.** Prefer changes that are easy to verify and easy to undo.
- **Make it work → make it right → make it fast,** in that order. Don't optimize what isn't correct yet.
- **Evidence over opinion.** Reproduce, measure, and check the data before deciding.
- **Deliver in small, working slices** over building the whole thing in the dark.
- **Write down decisions,** not just code. Future-you and your teammates need the *why*.
- **Leave it better than you found it,** but scope creep is the enemy — note it, don't chase it.

---

## Anti-Patterns to Catch Yourself On

- Jumping to a solution before the problem is understood (skipping Phase 1).
- "Done" with no definition of done — so it's never really done.
- Big-bang changes that can't be reviewed, tested, or undone in pieces.
- Debugging by guessing and changing many things at once.
- Fixing the symptom and moving on without finding the cause.
- Calling it done before it's verified, reviewed, and actually merged.
- Handing off with no rollback or deploy notes — leaving the shipper to guess.
- Carrying silent tech debt instead of logging it as a real follow-up.
- Gold-plating: polishing parts no user will notice while the core slips.

---

## One-Line Cheat Sheet

> **Understand the real problem → define done → choose the simplest correct approach → build it in small, verified slices → finalize a reviewed change that's ready for you to ship.**
