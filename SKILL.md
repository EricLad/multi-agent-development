---
name: multi-agent-development
description: Orchestrate Codex software development with progressive governance. Choose the lightest sufficient workflow: FAST for small low-risk edits handled directly by the main session, STANDARD for one-Developer work with inline exploration and optional review, and ORCHESTRATED for complex, high-risk, parallel, or hard-to-diagnose work using Explorer/Bug Investigator, worktrees, staging, commit-bound review, and integration gates. Use for feature work, bug fixes, refactors, performance work, maintenance, and multi-agent development.
---

# Multi-Agent Development

Use **progressive governance**: apply only the process whose safety benefit exceeds its coordination cost.

The default goal is not to maximize the number of agents or quality gates. The goal is to complete the change correctly with the **least orchestration necessary**.

## Core principles

1. **Exploration is required; a separate Explorer is optional.** Whoever implements the code should normally search, read, and understand the relevant code themselves.
2. **Prefer one context when one context is enough.** Do not split understanding and implementation across agents without a concrete orchestration or investigation benefit.
3. **Choose FAST, STANDARD, or ORCHESTRATED before starting implementation.** Start with the lightest tier that safely fits the task and upgrade only when evidence justifies it.
4. **Risk is not line count.** A tiny security, lifetime, concurrency, persistence, schema, protocol, or compatibility change may require stronger handling.
5. Read and obey repository `AGENTS.md`, build/test instructions, architecture rules, and user constraints in every tier.
6. Never overwrite, discard, reset, or silently absorb unrelated user changes.
7. Do not create branches, worktrees, subagents, reviews, staging branches, or bookkeeping fields merely for workflow uniformity.

# Workflow tiers

## FAST

Use FAST when all of the following are substantially true:

- the requested change is localized and easy to understand;
- one main-session coding context is sufficient;
- no task decomposition or parallel development is useful;
- risk is Low and the blast radius is small;
- validation is direct and targeted;
- there is no material security, concurrency/lifetime, persistent-data, schema/protocol, public-API, or broad compatibility concern;
- for a bug, the causal defect is obvious enough to diagnose while implementing.

Typical examples: a few local edits, a small UI/config change, a straightforward condition or field change, a small deterministic bug fix.

### FAST execution

The **main Codex session may implement production code directly**.

Canonical flow:

`understand request -> inspect git status/project rules -> search/read relevant code -> edit -> targeted build/test/check -> inspect final diff -> done`

FAST rules:

- exploration happens inline in the main coding context;
- no separate Explorer or Developer subagent;
- no independent Reviewer by default;
- no Agent Pool;
- no workflow-created worktree or staging branch;
- no mandatory temporary branch;
- no commit/SHA certification state machine;
- do not read heavy ORCHESTRATED references unless the task upgrades;
- use the project's normal validation commands, but keep validation proportional to the change.

If implementation reveals broader coupling, uncertain behavior, meaningful risk, or difficult diagnosis, **stop treating the task as FAST and upgrade to STANDARD or ORCHESTRATED**.

## STANDARD

Use STANDARD when the task is larger than a trivial local edit but is still best owned end-to-end by **one Developer**.

Typical signals:

- several related files or one subsystem are involved;
- moderate reasoning or regression surface exists;
- implementation benefits from delegation, but parallel decomposition would add coordination cost;
- a single Developer can safely explore, implement, and validate the change;
- risk is usually Low/Medium, though a bounded High-risk task may use STANDARD with stronger review/model handling when orchestration itself adds no value.

### STANDARD execution

Canonical flow:

`Controller gives concise goal/scope/acceptance -> one Developer explores + implements -> targeted validation -> optional independent Reviewer when justified -> fix/revalidate if needed -> Controller final check -> done`

STANDARD rules:

- **do not spawn an Explorer by default**;
- the Developer performs their own code search, local call-chain reading, implementation, and validation in the same context;
- use a **lightweight task brief**, not the full ORCHESTRATED contract unless needed;
- worktree/staging/Agent Pool are off by default;
- a temporary branch is optional and should follow repository/user practice, not workflow ceremony;
- the full commit-bound `TASK_HEAD/VALIDATED_HEAD/REVIEWED_HEAD/ACCEPTED_HEAD` state machine is optional, not mandatory;
- independent review is conditional, not automatic.

Request an independent Reviewer in STANDARD when any of these materially apply:

- Medium/High semantic risk;
- non-trivial public behavior/API compatibility;
- error-handling, persistence, security-sensitive, concurrency/lifetime, or other correctness-sensitive changes;
- the Developer expresses uncertainty;
- tests are weak or the regression surface is meaningful;
- the Controller cannot confidently validate the final diff itself.

For a small Low-risk STANDARD change with strong targeted validation, Controller diff inspection may replace a separate Reviewer.

If the task becomes multi-owner, dependency-heavy, hard to diagnose, broadly coupled, or high-blast-radius, upgrade to ORCHESTRATED.

## ORCHESTRATED

Use ORCHESTRATED only when the additional coordination and quality gates provide clear value.

Strong signals include:

- multiple Developers can safely work in parallel;
- the Controller needs a global dependency/hot-file map before decomposition;
- work crosses multiple coupled subsystems or shared interfaces;
- task ordering/base commits materially matter;
- the change is High/Critical risk or has large blast radius;
- the bug has unknown root cause, intermittent behavior, multiple plausible causes, concurrency/lifetime/state corruption, or difficult reproduction;
- final integration interactions need independent review;
- worktrees/staging materially reduce integration risk.

### Explorer policy

Do **not** use an Explorer merely because the task is called complex.

Use an Explorer when the Controller needs information to **orchestrate multiple tasks**, especially:

- ownership boundaries;
- hot files/shared APIs;
- dependencies and task order;
- safe parallelization groups;
- integration/validation ownership.

The Explorer should not duplicate the detailed implementation analysis that Developers will perform themselves. Read `references/explorer.md` only when an Explorer is actually justified.

### Bug Investigator policy

Use a separate Bug Investigator only when **diagnosis itself is a substantial independent problem**. Obvious/local bugs should be investigated by the same coding context that fixes them.

For difficult bug investigation, read `references/bugfix.md` and use the evidence/root-cause workflow there.

### ORCHESTRATED execution

When needed:

1. perform repository/Git preflight;
2. optionally run Explorer or Bug Investigator;
3. build the dependency/ownership plan;
4. create a workflow-owned staging branch;
5. create bounded task contracts and task branches/worktrees;
6. schedule Developers through the Agent Pool only when parallelism exists;
7. each Developer explores their owned surface, implements, commits, and performs scoped validation;
8. independently review task snapshots as required;
9. repair and re-review changed snapshots;
10. integrate accepted task snapshots into staging in dependency-safe order;
11. run final staging validation;
12. run Integration Review;
13. reconcile target drift and re-certify staging if needed;
14. promote the certified staging result to the user target;
15. clean all workflow-created Git resources safely.

For this tier, read the applicable heavy references:

- `references/code-state.md`
- `references/task-contract.md`
- `references/worker.md`
- `references/reviewer.md`
- `references/quality-and-git.md`
- `references/agent-pool.md` when more than one subagent may be active
- `references/explorer.md` only when orchestration exploration is needed
- `references/bugfix.md` only for difficult/non-obvious bug diagnosis
- `references/integration-reviewer.md`
- `references/model-routing.md` for delegated-role model selection/escalation

# Tier selection heuristics

Do not classify from file count or line count alone, but use scope as one signal.

Prefer **FAST** when a competent engineer would naturally make the change in one short coding pass and targeted validation is sufficient.

Prefer **STANDARD** when one engineer/agent should own the change end-to-end but the work benefits from a delegated implementation context or conditional review.

Prefer **ORCHESTRATED** when decomposition, parallelism, independent diagnosis, strict Git isolation, or integration governance materially reduces risk.

When uncertain between two tiers, choose the lighter tier unless a concrete risk signal justifies the heavier one. Upgrade during execution when new evidence appears.

# Model routing

Model routing applies to **delegated roles**, not as ceremony for FAST.

- FAST: use the current main-session model; do not create subagents merely to satisfy model-routing rules.
- STANDARD: route the single Developer by actual risk; use a Reviewer only when the review triggers above apply.
- ORCHESTRATED: use the full role+risk routing and escalation rules in `references/model-routing.md`.

Do not spend tokens recording `ROLE/RISK_LEVEL/ASSIGNED_MODEL/...` fields for FAST. In STANDARD, keep model/risk bookkeeping concise. Full auditable fields are primarily for ORCHESTRATED work.

# Bugfix routing

- obvious/local/deterministic bug -> FAST when safe;
- one-owner but non-trivial bug -> STANDARD, with the Developer investigating and fixing in one context;
- unknown/intermittent/cross-module/concurrency/lifetime/state-corruption bug -> ORCHESTRATED and consider a separate Bug Investigator.

Never confuse symptom suppression with root-cause correction. The amount of formal evidence should be proportional to uncertainty and risk.

# Review policy

Review intensity is proportional to risk:

- FAST: main-session final diff inspection + targeted validation by default;
- STANDARD: independent Reviewer when justified by risk/uncertainty, otherwise Controller final diff inspection;
- ORCHESTRATED: independent per-task review and final Integration Review are normal quality gates.

Do not require a fresh independent agent to re-read a tiny Low-risk diff solely because the Skill is active.

# Git policy

- FAST: do not create workflow Git resources by default; inspect status and preserve user work.
- STANDARD: branch/commit boundaries are optional unless repository policy, user request, or review safety requires them.
- ORCHESTRATED: use the Git resource ledger, worktrees, staging, commit-bound state, target-drift handling, and mandatory cleanup defined in the Git references.

# Completion

A task is complete when the requirements are met and the **applicable tier's** validation/review obligations are satisfied.

Do not apply ORCHESTRATED completion invariants to FAST/STANDARD work unless the workflow was explicitly upgraded or the user/project requires them.

In the final report, keep the process summary proportional to the task. Do not spend more tokens describing orchestration than the change warrants.