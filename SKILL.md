---
name: multi-agent-development
description: Orchestrate Codex software development with progressive governance. Choose the lightest sufficient workflow: FAST for small low-risk edits handled directly by the main session, STANDARD for one-Developer work with inline exploration and optional review, and ORCHESTRATED for complex, parallel, high-risk, or hard-to-diagnose work. Route models by execution ambiguity, task horizon, integration breadth, and cost instead of hardcoding one model for every delegated task.
---

# Multi-Agent Development

Use **progressive governance**: apply only the process whose safety benefit exceeds its coordination cost.

The goal is to complete the change correctly with the **least orchestration necessary** while preserving useful context and shortening the actual critical path.

## Core principles

1. **Exploration is required; a separate Explorer is optional.** Whoever implements the code should normally search, read, and understand the relevant code themselves.
2. **Prefer one context when one context is enough.** Strong long-horizon models make unnecessary handoffs more expensive, not more valuable.
3. **Choose FAST, STANDARD, or ORCHESTRATED before implementation.** Start with the lightest safe tier and upgrade only when evidence justifies it.
4. **Define outcomes and boundaries before delegation.** The Controller owns consequential architecture, public behavior, critical invariants, scope, and validation ownership; the Developer may choose routine local implementation tactics inside those boundaries.
5. **Route by capability, not model name alone.** Execution ambiguity, task horizon, integration breadth, expected iteration cost, and cost sensitivity determine model choice.
6. **Use cost-efficient models for bounded execution.** GPT-5.6 Terra remains a strong default for clear, plan-ready implementation where long-horizon reasoning is not the bottleneck.
7. **Use GPT-6 Astra where cognition dominates coordination.** Prefer Astra for long-horizon, cross-system, highly ambiguous, difficult integration, or repeatedly non-convergent work when available and justified.
8. **Escalate reasoning before multiplying handoffs.** When the runtime supports it, increase Astra reasoning effort within the same long-lived context before discarding useful state or spawning replacement agents solely for more reasoning power.
9. **Risk controls governance.** Security, concurrency/lifetime, persistence, schema/protocol, public compatibility, or large blast radius should strengthen validation/review even when a cost-efficient Developer remains sufficient.
10. **Batch expensive work.** Reviews, repairs, commits, and slow validation should be grouped when correctness permits.
11. **Converge, do not churn.** Repeated material review/repair failures trigger replanning rather than unbounded loops.
12. Read and obey repository `AGENTS.md`, build/test instructions, architecture rules, and user constraints in every tier.
13. Never overwrite, discard, reset, or silently absorb unrelated user changes.
14. Do not create branches, worktrees, subagents, reviews, staging branches, or bookkeeping fields merely for workflow uniformity.

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
- once required targeted checks pass, do not broaden or repeat validation unless later changes, failures, or concrete unresolved risk invalidate that evidence;
- do not load heavy ORCHESTRATED references unless the task upgrades.

If implementation reveals broader coupling, consequential ambiguity, meaningful risk, or difficult diagnosis, upgrade to STANDARD or ORCHESTRATED.

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
3. **Architectural decisions / hard constraints** — only decisions the Developer must not silently change;
4. **Non-goals** — nearby work that must not be changed;
5. **Validation** — concrete success checks.

The Controller does **not** need to prescribe routine local implementation tactics when repository evidence can resolve them safely. A Developer may choose helpers, file-local structure, exact edit sequence, and similarly reversible details inside the accepted boundary.

If these fields are clear, treat the task as `PLAN_READY` and route the Developer by capability/cost rather than by risk alone.

### STANDARD execution

Canonical flow:

`Controller makes boundary ready -> one Developer explores local code + implements -> targeted validation -> optional independent Reviewer when justified -> grouped repair/revalidation if needed -> Controller final check -> done`

STANDARD rules:

- **do not spawn an Explorer by default**;
- preserve one Developer context for local search, call/data-flow reading, implementation, repair, and validation when practical;
- use the lightweight task brief from `references/task-contract.md`;
- worktree/staging/Agent Pool are off by default;
- temporary branch and full commit-bound SHA state are optional;
- independent review is conditional, not automatic;
- when review is used, prefer one complete review and one grouped repair rather than serial one-finding cycles.

Request an independent Reviewer when risk/uncertainty materially justifies it, including public behavior/API compatibility, persistence, security, concurrency/lifetime, weak tests, meaningful regression surface, or Developer uncertainty.

If a consequential plan assumption is false, block only the dependent slice when safe; independent work that does not rely on that decision may continue. Return the decision to the Controller instead of silently changing architecture or expanding scope.

If the task becomes multi-owner, dependency-heavy, hard to diagnose, broadly coupled, or high-blast-radius, upgrade to ORCHESTRATED.

## ORCHESTRATED

Use ORCHESTRATED only when decomposition, parallelism, independent diagnosis, strict Git isolation, or integration governance provides clear value.

Strong signals include:

- multiple Developers can safely work in parallel and parallelism shortens the critical path;
- a global dependency/hot-file map is needed before decomposition;
- work crosses coupled subsystems/shared interfaces;
- task ordering/base commits matter;
- the change has large blast radius or High/Critical governance risk;
- a Bug has unknown root cause, intermittent behavior, concurrency/lifetime/state corruption, or difficult reproduction;
- final integration interactions need independent review.

### Explorer policy

Do **not** use an Explorer merely because the task is called complex.

Use an Explorer when the Controller needs information to orchestrate multiple tasks: ownership boundaries, hot files, dependencies, safe parallel groups, **critical path**, and integration/validation ownership.

Explorer output should make decomposition clearer; it should not duplicate the detailed implementation analysis Developers will do locally.

### Bug Investigator policy

Use a separate Bug Investigator only when diagnosis itself is a substantial independent problem. Obvious/local bugs should be investigated by the same coding context that fixes them.

### Parallel-delegation trigger

When **two or more independent Ready tasks** lie on, unblock, or materially shorten the critical path, prefer parallel delegation if expected wall-clock savings exceed coordination, review, build, and merge costs.

Do not rely on a model to volunteer parallelism. The Controller should explicitly delegate when this trigger is met, while avoiding hot-file conflicts and speculative fan-out.

### ORCHESTRATED planning and execution

1. perform repository/Git preflight;
2. optionally run Explorer or Bug Investigator;
3. build dependency/ownership, critical-path, and risk plans;
4. create a workflow-owned staging branch;
5. decompose work into bounded tasks;
6. for **each Developer task**, pass the Plan Readiness Gate; for High/Critical tasks state only the Critical Invariants that materially constrain correctness;
7. create task branches/worktrees and schedule Developers through Agent Pool only when parallelism shortens the critical path;
8. route each Developer using `references/model-routing.md`;
9. each Developer verifies local assumptions, implements narrowly, commits in meaningful batches, and performs scoped validation;
10. preserve the Developer context for compatible repair work when practical instead of creating a fresh agent solely because a review found issues;
11. run one **complete batch review** per task snapshot; collect material findings before returning them;
12. route the finding batch to the Developer for one **batch repair** when practical, then revalidate and re-review the final repaired snapshot;
13. if a second material review/repair cycle still produces substantial new findings, run a **Convergence Gate** before continuing;
14. integrate accepted task snapshots into staging in dependency-safe order;
15. run final staging validation using the Validation Pyramid and then Integration Review;
16. batch integration findings/repairs when possible; run Convergence Gate if integration review churns;
17. reconcile target drift and re-certify staging if needed;
18. promote the certified staging result to the user target;
19. clean workflow-created Git resources safely.

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

A task is ready when the Developer knows **what outcome is required, what boundary must be respected, and how success will be verified** without needing to make a consequential architectural/public-contract decision.

Minimum fields:

- Goal;
- Scope;
- Architectural decisions / hard constraints, if any;
- Non-goals;
- Validation.

For High/Critical ORCHESTRATED work, add only the **Critical Invariants** that materially constrain correctness, normally no more than 5-10 concise items.

Do not over-specify routine implementation tactics that a capable Developer can resolve from repository conventions. Excessive pre-design duplicates work, increases token cost, and can prevent the implementation agent from using better local evidence.

For ORCHESTRATED work, also include dependencies/base/integration fields and important permissions or risk constraints.

If the boundary is not ready:

`Controller/repository analysis -> optional Explorer/Bug Investigator -> resolve consequential decisions -> PLAN_READY -> Developer`

Do not use:

`unclear consequential boundary -> Developer silently redesigns architecture -> larger/uncontrolled diff`

# Capability and model routing

Read `references/model-routing.md` for the detailed policy.

Use these routing dimensions:

- `EXECUTION_AMBIGUITY` — how much consequential reasoning remains inside implementation;
- `TASK_HORIZON` — Short / Medium / Long;
- `INTEGRATION_BREADTH` — Local / Subsystem / Cross-system;
- `COST_SENSITIVITY` — Low / Medium / High when it materially affects model choice;
- `RISK_LEVEL` — controls governance/review strength, not Developer intelligence by itself.

Default tendencies when models are available:

- **bounded, plan-ready routine implementation** -> GPT-5.6 Terra medium;
- **long-horizon, cross-system, materially ambiguous implementation** -> GPT-6 Astra medium/high;
- **difficult unresolved semantics or repeated non-convergence** -> Astra high/xhigh/max as justified;
- **cost-sensitive exploration/review** -> use lower-cost capable models when the evidence task is bounded;
- **Astra unavailable** -> fall back to the strongest available GPT-5.6 path appropriate to the same capability requirement.

Do not replace a good Terra path merely because Astra exists. Do not keep a weak model on a task whose dominant cost is repeated reasoning failure and rework.

# Context continuity policy

For long-running STANDARD or ORCHESTRATED work, preserving useful state can be more valuable than creating a fresh agent.

When the runtime provides long-context/context-management/searchable-history capabilities:

- prefer a long-lived Controller for orchestration state;
- prefer keeping one Developer on implementation plus compatible repair work;
- avoid repeating full Explorer/task context when the active agent can retrieve or already retains it;
- summarize only stable boundaries, decisions, invariants, and handoff facts rather than duplicating full history;
- split contexts for ownership/isolation/parallelism/review independence, not merely because the task has become long.

Context continuity never overrides Git isolation, read-only review boundaries, independent-review value, or explicit task ownership.

# Developer blocked-plan rule

A Developer must not silently turn an implementation task into a consequential design task.

When a key assumption is false:

1. identify the dependent slice that cannot proceed safely;
2. continue independent, reversible work only when it does not prejudge the decision;
3. return or asynchronously ask for the smallest consequential decision needed when the runtime supports that interaction;
4. resume the blocked slice after the Controller resolves the boundary.

Report:

- Expected;
- Actual;
- Impact;
- Blocked slice;
- Independent work that can safely continue, if any;
- Decision needed.

Do not stall the entire task for routine local choices that repository conventions can safely resolve.

# Validation Pyramid

Validation should increase in breadth as code approaches integration.

## Inner loop

During implementation and repair, run the fastest checks that directly exercise the changed surface:

`affected target compile -> directly relevant tests/checks`

Do not run the full project suite after every small edit unless the project makes that genuinely cheap or the failure mode requires it.

Once the required targeted checks pass, stop expanding or repeating validation unless new changes, failures, or concrete unresolved risk invalidate that evidence.

## Task gate

Before authoritative ORCHESTRATED task review/acceptance, validate the final committed `TASK_HEAD` with the task-specific build/test suite required by its contract.

## Staging gate

After accepted tasks are integrated, run the expensive broad checks once on the integrated staging snapshot: clean/fresh build as appropriate, full project tests, integration/regression/stress/real-data checks required by the project.

If a later repair changes relevant semantics, rerun the invalidated level. Do not automatically rerun unrelated expensive layers.

# Batch Review and Repair

For a reviewable snapshot, the Reviewer should normally perform a complete pass and return the material finding set together.

Do not intentionally stop after the first non-BLOCKER issue. Group compatible findings into one Developer repair batch, one validation pass, and one re-review where practical.

Classify findings by both severity and disposition:

- **Required Defect** — violates requirements, stated invariants, compatibility, or established correctness expectations; blocks according to severity;
- **Contract Gap** — exposes an important missing/ambiguous requirement or invariant; Controller decides/replans before dependent implementation continues;
- **Optional Hardening** — defensive improvement beyond the accepted contract; normally does not block;
- **Deferred** — valid future work intentionally left outside the current task.

Review is for finding defects in the accepted contract, not for endlessly expanding that contract.

# Convergence Gate

Default target for ORCHESTRATED work:

`implementation -> complete review -> one batch repair -> revalidation/re-review -> converge`

A second material repair/re-review cycle is allowed when justified, but if substantial new BLOCKER/HIGH/MEDIUM issues continue to appear, pause normal cycling and ask:

- Is the implementation approach wrong or incomplete?
- Are Critical Invariants missing?
- Is the task boundary too broad or incorrectly split?
- Is the Reviewer discovering new requirements rather than defects?
- Is validation exposing a deeper root cause or architecture problem?
- Is the current model/reasoning effort causing repeated reasoning failure that justifies capability escalation?

The Controller must replan, split, defer optional hardening, increase reasoning capability, or explicitly authorize another cycle. Do not continue an unbounded fix/test/review loop by default.

# Bugfix routing

- obvious/local/deterministic Bug -> FAST when safe;
- one-owner non-trivial Bug -> STANDARD, same Developer investigates and fixes;
- unknown/intermittent/cross-module/concurrency/lifetime/state-corruption Bug -> ORCHESTRATED and consider Bug Investigator;
- if diagnosis remains long-horizon or multi-hypothesis after bounded investigation, prefer Astra-class reasoning when available instead of serially spawning more weak diagnosis contexts.

Never confuse symptom suppression with root-cause correction. Formal evidence should be proportional to uncertainty and risk.

# Review policy

Review intensity is proportional to risk and uncertainty:

- FAST: main-session diff inspection + targeted validation by default;
- STANDARD: independent Reviewer when justified, otherwise Controller final diff inspection;
- ORCHESTRATED: independent per-task review and final Integration Review are normal gates.

Reviewer capability should match defect-finding difficulty and integration breadth, not merely the Developer model name.

# Git policy

- FAST: do not create workflow Git resources by default; preserve user work.
- STANDARD: branch/commit boundaries are optional unless project/user/review safety requires them.
- ORCHESTRATED: use Git resource ledger, worktrees, staging, commit-bound state, target-drift handling, and mandatory cleanup.
- Prefer **meaningful commit batches** over one commit per tiny finding/test when the repository/user policy permits it.

# Lightweight workflow metrics

For long ORCHESTRATED tasks, the Controller should keep lightweight in-memory counters when practical:

- approximate planning time;
- implementation time;
- review time;
- repair time;
- validation/integration time;
- number of Developers;
- number of material review/repair cycles;
- model/reasoning escalations that materially affected convergence.

Do not create extra agents, repository files, or verbose logs solely for metrics. A concise final summary is enough. These metrics exist to identify future workflow bottlenecks.

# Completion

A task is complete when requirements are met and the applicable tier's validation/review obligations are satisfied.

Do not apply ORCHESTRATED completion invariants to FAST/STANDARD unless the workflow was explicitly upgraded or the user/project requires them.

Keep final reporting proportional to the task.
