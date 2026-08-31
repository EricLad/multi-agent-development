---
name: multi-agent-development
description: Orchestrate Codex software development with progressive governance. Choose the lightest sufficient workflow: FAST for small low-risk edits handled directly by the main session, STANDARD for one-Developer work with inline exploration and optional review, and ORCHESTRATED for complex, parallel, high-risk, or hard-to-diagnose work. Delegated Developers default to GPT-5.6 Terra medium once the Controller has made the implementation plan explicit.
---

# Multi-Agent Development

Use **progressive governance**: apply only the process whose safety benefit exceeds its coordination cost.

The goal is to complete the change correctly with the **least orchestration necessary**.

## Core principles

1. **Exploration is required; a separate Explorer is optional.** Whoever implements the code should normally search, read, and understand the relevant code themselves.
2. **Prefer one context when one context is enough.** Do not split understanding and implementation across agents without concrete orchestration or investigation value.
3. **Choose FAST, STANDARD, or ORCHESTRATED before implementation.** Start with the lightest safe tier and upgrade only when evidence justifies it.
4. **Plan before delegation.** A Developer should receive an implementation-ready plan instead of being asked to rediscover architecture or invent the solution.
5. **Developer = execution agent.** For delegated implementation, prefer GPT-5.6 Terra medium when the plan is explicit. Stronger Developer models are for unresolved execution ambiguity, not simply higher risk.
6. **Risk controls governance.** Security, concurrency/lifetime, persistence, schema/protocol, public compatibility, or large blast radius should strengthen validation/review even when Terra medium remains sufficient to implement a clear plan.
7. Read and obey repository `AGENTS.md`, build/test instructions, architecture rules, and user constraints in every tier.
8. Never overwrite, discard, reset, or silently absorb unrelated user changes.
9. Do not create branches, worktrees, subagents, reviews, staging branches, or bookkeeping fields merely for workflow uniformity.

# Workflow tiers

## FAST

Use FAST when the change is localized, easy to understand, Low risk, directly verifiable, and one main-session coding context is sufficient.

Typical examples: a few local edits, a small UI/config change, a straightforward condition or field change, or a deterministic small Bugfix.

### FAST execution

The **main Codex session may implement production code directly**.

Canonical flow:

`understand request -> inspect git status/project rules -> search/read relevant code -> edit -> targeted build/test/check -> inspect final diff -> done`

FAST rules:

- exploration happens inline in the main coding context;
- no separate Explorer or Developer subagent;
- no independent Reviewer by default;
- no Agent Pool, worktree, staging branch, mandatory temporary branch, or commit/SHA state machine;
- keep validation proportional to the change;
- do not load heavy ORCHESTRATED references unless the task upgrades.

If implementation reveals broader coupling, uncertain behavior, meaningful risk, or difficult diagnosis, upgrade to STANDARD or ORCHESTRATED.

## STANDARD

Use STANDARD when one Developer should own the change end-to-end but delegation is still useful.

Typical signals:

- several related files or one subsystem are involved;
- a moderate reasoning/regression surface exists;
- parallel decomposition would add more coordination than value;
- one Developer can explore, implement, and validate safely.

### Plan Readiness Gate

Before starting the Developer, ensure these are clear enough:

1. **Goal** — required outcome;
2. **Scope** — owned subsystem/files/behavior;
3. **Implementation approach** — intended solution and important constraints;
4. **Non-goals** — nearby work that must not be changed;
5. **Validation** — concrete success checks.

If they are clear, treat the task as `PLAN_READY` and use **GPT-5.6 Terra medium** by default.

If they are not clear, improve the plan first. Do not use a stronger Developer model merely to compensate for an unclear task.

### STANDARD execution

Canonical flow:

`Controller makes plan ready -> Terra medium Developer explores local code + implements -> targeted validation -> optional independent Reviewer when justified -> fix/revalidate if needed -> Controller final check -> done`

STANDARD rules:

- **do not spawn an Explorer by default**;
- the Developer performs local code search, call/data-flow reading, implementation, and validation in the same context;
- use the lightweight task brief from `references/task-contract.md`;
- worktree/staging/Agent Pool are off by default;
- temporary branch and full commit-bound SHA state are optional;
- independent review is conditional, not automatic.

Request an independent Reviewer when risk/uncertainty materially justifies it, including public behavior/API compatibility, persistence, security, concurrency/lifetime, weak tests, meaningful regression surface, or Developer uncertainty.

If the Developer discovers that a key plan assumption is false, it must return the issue to the Controller rather than silently redesigning or expanding scope.

If the task becomes multi-owner, dependency-heavy, hard to diagnose, broadly coupled, or high-blast-radius, upgrade to ORCHESTRATED.

## ORCHESTRATED

Use ORCHESTRATED only when decomposition, parallelism, independent diagnosis, strict Git isolation, or integration governance provides clear value.

Strong signals include:

- multiple Developers can safely work in parallel;
- a global dependency/hot-file map is needed before decomposition;
- work crosses coupled subsystems/shared interfaces;
- task ordering/base commits matter;
- the change has large blast radius or High/Critical governance risk;
- a Bug has unknown root cause, intermittent behavior, concurrency/lifetime/state corruption, or difficult reproduction;
- final integration interactions need independent review.

### Explorer policy

Do **not** use an Explorer merely because the task is called complex.

Use an Explorer when the Controller needs information to orchestrate multiple tasks: ownership boundaries, hot files, dependencies, safe parallel groups, and integration/validation ownership.

Explorer output should make decomposition clearer; it should not duplicate the detailed implementation analysis Developers will do locally.

### Bug Investigator policy

Use a separate Bug Investigator only when diagnosis itself is a substantial independent problem. Obvious/local bugs should be investigated by the same coding context that fixes them.

### ORCHESTRATED planning and execution

1. perform repository/Git preflight;
2. optionally run Explorer or Bug Investigator;
3. build dependency/ownership and risk plans;
4. create a workflow-owned staging branch;
5. decompose work into bounded tasks;
6. for **each Developer task**, pass the Plan Readiness Gate before starting it;
7. create task branches/worktrees and schedule Developers through Agent Pool only when parallelism exists;
8. use **GPT-5.6 Terra medium as the default Developer** for plan-ready tasks;
9. each Developer verifies local assumptions, implements narrowly, commits, and performs scoped validation;
10. independently review task snapshots as required;
11. repair and re-review changed snapshots;
12. integrate accepted task snapshots into staging in dependency-safe order;
13. run final staging validation and Integration Review;
14. reconcile target drift and re-certify staging if needed;
15. promote the certified staging result to the user target;
16. clean workflow-created Git resources safely.

For ORCHESTRATED work, read applicable references:

- `references/code-state.md`
- `references/task-contract.md`
- `references/worker.md`
- `references/reviewer.md`
- `references/quality-and-git.md`
- `references/agent-pool.md` when multiple subagents may be active
- `references/explorer.md` only when orchestration exploration is justified
- `references/bugfix.md` only for difficult/non-obvious Bugfix diagnosis
- `references/integration-reviewer.md`
- `references/model-routing.md`

# Plan Readiness Gate

The Plan Readiness Gate is the main quality/cost control before delegated implementation.

A task is ready when the Developer can answer **what to change and how to verify it** without having to make a new architectural decision.

Minimum fields:

- Goal;
- Scope;
- Implementation approach;
- Non-goals;
- Validation.

For ORCHESTRATED work, also include dependencies/base/integration fields and any important permissions or risk constraints.

If the plan is not ready:

`Controller/repository analysis -> optional Explorer/Bug Investigator -> clarify solution -> PLAN_READY -> Developer`

Do not use:

`unclear plan -> stronger Developer guesses -> larger/uncontrolled diff`

# Developer model routing

Developer routing is based primarily on **execution ambiguity**, not task risk.

## Default

`PLAN_READY -> GPT-5.6 Terra medium`

Use Terra medium for ordinary production work, bounded multi-file changes, clear ORCHESTRATED subtasks, and even high-risk tasks when the implementation strategy is already explicit.

## Escalate to Terra xhigh

Escalate only when unresolved implementation reasoning remains, such as:

- the contract conflicts materially with actual architecture;
- multiple materially different implementation choices remain;
- expected API/ownership/lifecycle assumptions are false;
- repeated build/test failures do not converge;
- unexpected cross-module coupling requires reasoning beyond a small Controller replan.

Prefer replanning first when the decision belongs to the Controller.

## Escalate to Terra max / Sol

Use Terra max only for genuinely difficult implementation semantics such as unresolved concurrency/lifetime/state-machine interactions or similarly subtle correctness problems.

Reserve Sol xhigh/max for Critical or repeatedly unresolved implementation work.

Do not escalate Developer solely because `RISK_LEVEL` is High. Strengthen review/validation instead when the plan itself is clear.

# Other role model routing

Read `references/model-routing.md` when delegated roles are used.

Defaults:

- Explorer -> Luna max;
- Bug Investigator -> Luna max / Terra xhigh / Terra max according to diagnosis difficulty;
- Developer -> **Terra medium** when plan-ready;
- Reviewer -> Luna max, escalating to Terra xhigh for high-risk/uncertain review;
- Integration Reviewer -> Terra xhigh, escalating to Terra max for difficult/high-risk integration;
- Sol -> Critical/repeatedly unresolved work.

# Developer blocked-plan rule

A Developer must not silently turn an implementation task into a design task.

When a key assumption is false, return:

- Expected;
- Actual;
- Impact;
- Decision needed.

The Controller replans. The task may then continue with Terra medium if the new plan is explicit.

# Bugfix routing

- obvious/local/deterministic Bug -> FAST when safe;
- one-owner non-trivial Bug -> STANDARD, same Developer investigates and fixes;
- unknown/intermittent/cross-module/concurrency/lifetime/state-corruption Bug -> ORCHESTRATED and consider Bug Investigator.

Never confuse symptom suppression with root-cause correction. Formal evidence should be proportional to uncertainty and risk.

# Review policy

Review intensity is proportional to risk and uncertainty:

- FAST: main-session diff inspection + targeted validation by default;
- STANDARD: independent Reviewer when justified, otherwise Controller final diff inspection;
- ORCHESTRATED: independent per-task review and final Integration Review are normal gates.

A Terra medium Developer does **not** automatically require a stronger Reviewer. Review tier depends on the defect-finding difficulty and risk of the change.

# Git policy

- FAST: do not create workflow Git resources by default; preserve user work.
- STANDARD: branch/commit boundaries are optional unless project/user/review safety requires them.
- ORCHESTRATED: use Git resource ledger, worktrees, staging, commit-bound state, target-drift handling, and mandatory cleanup.

# Completion

A task is complete when requirements are met and the applicable tier's validation/review obligations are satisfied.

Do not apply ORCHESTRATED completion invariants to FAST/STANDARD unless the workflow was explicitly upgraded or the user/project requires them.

Keep final reporting proportional to the task.