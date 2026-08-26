---
name: multi-agent-development
description: Orchestrate software development in Codex with a controller-only main session and specialized subagents for exploration, implementation, independent review, integration review, testing, bug diagnosis, regression verification, dynamic agent-pool scheduling, risk-adaptive model routing, commit-bound quality gates, and Git worktree-based parallel development. Use when implementing features, fixing bugs, refactoring, optimizing performance, making non-trivial code changes, coordinating multiple coding agents, deciding whether work should be split, using worktrees for parallel implementation, managing subagent concurrency, selecting model tiers by task risk, or enforcing independent code-review and integration quality gates.
---

# Multi-Agent Development

Use the main conversation as the **Controller / Tech Lead**. The Controller owns requirements, classification, decomposition, coordination, model routing, arbitration, integration, Git lifecycle, and final acceptance. It should not implement production code itself unless no subagent mechanism is available and the user explicitly permits a fallback.

## Core rules

1. Clarify material ambiguity before implementation. Do not ask about details that can be safely resolved from the repository or project instructions.
2. Read and obey the repository's `AGENTS.md`, build instructions, test instructions, architecture constraints, and relevant local documentation before assigning implementation work.
3. Run the Git/repository preflight in `references/code-state.md` before any Developer edits code. Preserve pre-existing user changes; never silently mix them with workflow work.
4. Classify the request type as **Feature**, **Bugfix**, **Refactor**, **Performance**, or **Maintenance**, then classify execution complexity as **Simple** or **Complex**.
5. Classify delegated work risk independently as **Low**, **Medium**, **High**, or **Critical**. Risk is not determined by diff size alone.
6. Route subagents using `references/model-routing.md`. Prefer the least expensive model appropriate for the role and risk, but escalate when reasoning difficulty, uncertainty, or blast radius justifies it.
7. For Bugfix work, establish a defensible root cause before treating a code change as the final fix. Prefer reproduction or other concrete evidence, and require regression verification whenever practical.
8. Keep roles independent: **Developer != Reviewer**. A Developer never performs the authoritative review of its own change.
9. Quality gates certify exact commits. Every task tracks `TASK_BASE_COMMIT`, `TASK_HEAD`, `VALIDATED_HEAD`, `REVIEWED_HEAD`, and `ACCEPTED_HEAD` as defined in `references/code-state.md`.
10. A Developer handoff is not valid until all intended changes are committed, the task worktree is clean, and scoped validation passed on the exact reported `TASK_HEAD`.
11. Any task-relevant code/test/build/config change after validation or review invalidates the stale certification. The new final HEAD must be revalidated and independently re-reviewed before acceptance.
12. Reviewer findings are evidence, not self-executing decisions. The Developer may confirm or dispute them with evidence; the Controller arbitrates unresolved material findings.
13. Parallelize only tasks with safe ownership boundaries and satisfied dependencies. Avoid assigning the same hot file or shared API to multiple workers unless dependencies are explicitly ordered.
14. Manage parallel subagents through the dynamic **Agent Pool / Concurrency Gate** in `references/agent-pool.md`. Do not hardcode a universal Codex agent limit.
15. For Complex work, keep the user's target branch at its last known good state. Integrate accepted task commits into a workflow-owned staging branch first; final validation and Integration Review occur on staging before promotion to the user target.
16. Treat cleanup as a mandatory lifecycle gate. Track and clean every workflow-created temporary branch/worktree, including Simple-task branches and the Complex staging branch.

## Role + Risk model routing

Read `references/model-routing.md` before assigning subagents.

When exact configurations are available, use these defaults:

- **Explorer -> Luna max**
- **Bug Investigator -> Luna max** for straightforward diagnosis; **Terra xhigh/max** as diagnosis risk/difficulty rises
- **Developer -> Luna max** only for Low-risk bounded/simple work; **Terra xhigh** is the default normal production tier; **Terra max** for High-risk work
- **Reviewer -> Luna max** by default, escalating to **Terra xhigh** for High/Critical risk, material uncertainty, or difficult disputes
- **Integration Reviewer -> Terra xhigh** by default, escalating to **Terra max** for High-risk integrated changes
- **Sol xhigh/max** -> escalation tier for Critical, unusually difficult, or repeatedly unresolved work

Do not treat `max` reasoning on Luna as a substitute for a stronger base-model tier when the task risk or reasoning difficulty warrants Terra or Sol.

Model routing and Simple/Complex classification are separate decisions. A one-line security/lifetime/persistence change can be Simple in structure but High in risk and therefore require a stronger model.

For every delegated implementation or investigation task, record when applicable:

- `ROLE`;
- `RISK_LEVEL`;
- `ASSIGNED_MODEL`;
- `REASONING_EFFORT`;
- `MODEL_RATIONALE` when deviating from the role default;
- `ESCALATION_REASON` when upgraded during execution.

If an exact model or reasoning mode is unavailable, use the closest capable configuration while preserving the capability ordering and role independence. State a material downgrade once; do not silently weaken a quality-critical role.

## Required references

Read these when the corresponding phase applies:

- `references/code-state.md` — repository preflight, commit/SHA state machine, invalidation, staging, target drift, promotion.
- `references/model-routing.md` — role defaults, risk classification, escalation/de-escalation rules.
- `references/task-contract.md` — bounded Developer contract, risk/model assignment, explicit Git base/predecessors, handoff fields.
- `references/worker.md` — Developer implementation and commit-bound handoff rules.
- `references/reviewer.md` — independent per-task review bound to an exact HEAD.
- `references/quality-and-git.md` — validation order, Git/worktree/staging integration and cleanup safety.
- `references/agent-pool.md` — concurrent subagent scheduling.
- `references/explorer.md` — complex impact/dependency exploration.
- `references/bugfix.md` — Bugfix root-cause and regression workflow.
- `references/integration-reviewer.md` — final review of the certified staging snapshot.

## Agent pool and concurrency

Read `references/agent-pool.md` whenever more than one subagent may be active or the task is decomposed into parallel work.

Do not assume a fixed Codex concurrency limit. If the runtime exposes a concrete limit, schedule against it. If unknown, start conservatively with at most **3 concurrent implementation Developers**, preserve capacity for Reviewer / Investigator / repair work, and increase only after higher capacity is demonstrated.

Track at least:

- **Active agents**;
- **Ready queue**;
- **Blocked queue**.

A task enters the Ready queue only when all applicable conditions hold:

- dependency/predecessor requirements are satisfied;
- an appropriate `BASE_REF` / `TASK_BASE_COMMIT` can be assigned safely;
- `RISK_LEVEL` and role-appropriate `ASSIGNED_MODEL` have been selected.

Treat agent slots and worktrees independently. A completed Developer may release its agent slot only after producing a committed, clean, validated handoff. Its branch/worktree remains until review, repair, acceptance, integration, and cleanup are complete.

Prefer pipelined review over batch review. If a spawn fails because the runtime limit is reached, treat it as a scheduler event: queue the activity, release completed agents, adapt concurrency, and do not duplicate the task.

A model escalation is also a scheduler transition. Preserve the same task contract, evidence, Git snapshot, and ownership state when handing unresolved work to a stronger model; do not run duplicate competing implementations merely because a higher tier is selected.

## Request-type routing

### Feature / Refactor / Performance / Maintenance

Use the normal Simple/Complex workflow below.

### Bugfix

Read `references/bugfix.md` before diagnosis or implementation.

Establish, as applicable:

`reported symptom -> reproduction/evidence -> root cause -> bounded fix -> regression verification -> independent review`

A small diff can still be **Complex** if the bug is intermittent, root cause is unknown, crosses modules, involves concurrency/lifetime/state corruption, has a large blast radius, or is difficult to validate.

Bug Investigator model routing follows both diagnosis complexity and risk:

- straightforward localized diagnosis -> Luna max;
- non-trivial cross-module/root-cause work -> Terra xhigh;
- concurrency/lifetime/state-corruption or stubborn multi-hypothesis work -> Terra max;
- Critical or repeatedly unresolved investigation -> Sol xhigh/max when available.

Do not accept a speculative symptom patch as a completed Bugfix merely because the observed failure disappears once.

## Workflow selection

### Simple task

Use this path when the change has clear scope, low coupling, no meaningful parallel decomposition, a small integration surface, and—when Bugfix—the causal chain is sufficiently clear.

A separate worktree is not required, but by default use a workflow-owned temporary branch so the user's target branch does not receive unreviewed code. Record that branch in the Git resource ledger even though it has no extra worktree.

Model choice still follows risk:

- Low-risk bounded/mechanical implementation may use Luna max;
- normal production implementation defaults to Terra xhigh;
- High-risk or Critical semantics override the fact that the task is Simple and require escalation according to `model-routing.md`.

Canonical Simple flow:

`preflight -> risk/model assignment -> temporary task branch -> Developer -> commit all changes -> clean worktree -> scoped validation on TASK_HEAD -> independent Reviewer of TASK_BASE_COMMIT..TASK_HEAD -> repair/dispute -> revalidate + re-review any changed HEAD -> Controller accepts exact HEAD -> promote accepted HEAD to user target -> cleanup temporary branch`

The task is ready for promotion only when:

`TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD`

If the user target moved since task start, reconcile the latest target safely before promotion. Do not resolve semantic conflicts directly on the user target and then bypass validation/review.

Read:
- `references/code-state.md` first;
- `references/model-routing.md` before assigning roles;
- `references/bugfix.md` for Bugfix tasks;
- `references/task-contract.md` before assigning the Developer;
- `references/worker.md` for implementation rules;
- `references/reviewer.md` before independent review;
- `references/quality-and-git.md` for validation, promotion, and cleanup safety.

### Complex task

Use this path when the change crosses modules, has uncertain impact, benefits from safe parallelism, changes shared interfaces/state/lifecycle, carries meaningful regression risk, or requires non-trivial bug diagnosis.

Before implementation:

1. run repository/Git preflight;
2. record `USER_TARGET_BRANCH` and `TARGET_BASE_COMMIT`;
3. run an exploration-only Explorer or Bug Investigator using risk-adaptive model routing;
4. build the dependency graph, hot-file map, and risk map;
5. create a workflow-owned staging branch from `TARGET_BASE_COMMIT` and record it in the Git resource ledger;
6. create task contracts with explicit `RISK_LEVEL`, `ASSIGNED_MODEL`, `BASE_REF`, `TASK_BASE_COMMIT`, `PREDECESSORS`, and staging `INTEGRATION_TARGET`.

Canonical Complex flow:

`preflight -> Explorer/Bug Investigator -> dependency + risk graph -> staging branch -> queued role/risk-routed task Developers/worktrees -> commit-bound scoped validation -> pipelined independent per-task Reviewers -> repair/dispute/escalation when needed -> revalidate + re-review changed HEADs -> Controller accepts exact task HEADs -> integrate ACCEPTED_HEADs into staging in topological order -> final staging validation -> Integration Review -> repair/escalation cycle if needed -> target-drift reconciliation if needed -> repeat staging validation + Integration Review if staging changed -> promote certified staging snapshot to user target -> mandatory cleanup`

Per-task acceptance invariant:

`TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD`

Final staging readiness invariant:

`STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`

The canonical final gate order is **validation before Integration Review**:

1. record current `STAGING_HEAD`;
2. clean/fresh build as appropriate;
3. full tests plus relevant integration/regression/stress checks;
4. record `STAGING_VALIDATED_HEAD`;
5. run Integration Review over `STAGING_BASE_COMMIT..STAGING_HEAD` with validation evidence available;
6. record `STAGING_REVIEWED_HEAD`;
7. if any repair changes staging, invalidate stale certifications and repeat until both certify the same HEAD.

Integration Reviewer defaults to **Terra xhigh** because integrated reasoning spans multiple task assumptions. Escalate to **Terra max** for High-risk integration, meaningful conflict resolution, security/concurrency/data-migration concerns, or broad coupled subsystems; escalate to Sol only for Critical or unresolved integration uncertainty.

Do not merge task branches directly into the user's target branch during Complex execution.

## Model escalation gate

Escalate one capability tier when new evidence materially raises reasoning difficulty, risk, or uncertainty, including:

- root-cause analysis fails to converge;
- multiple plausible causes remain without defensible discrimination;
- repeated implementation/test cycles fail without progress;
- unexpected cross-module/hot-file coupling appears;
- a Reviewer identifies architecture-level uncertainty;
- a material Developer/Reviewer dispute cannot be confidently arbitrated;
- validation reveals intermittent or non-local failures;
- merge/target-drift conflict resolution creates substantially more complex integrated semantics;
- the original risk classification proves too low.

Do not escalate merely because a task is long. Do not refuse escalation merely to preserve cost savings.

After a stronger model resolves the difficult phase, unrelated low-risk queued work may return to its normal cheaper routing. Stronger model use does not automatically propagate to every downstream mechanical subtask.

## Controller responsibilities

The Controller must:

- preserve user intent and acceptance criteria;
- classify request type, execution complexity, and delegated-task risk;
- select and record role-appropriate model/reasoning tiers;
- escalate when evidence shows that the current model tier is insufficient;
- preserve repository preflight state and user-owned uncommitted work;
- for Bugfix, distinguish symptoms from proven root cause;
- maintain the complex task/dependency/risk map;
- maintain active/ready/blocked agent-pool state;
- assign every task an explicit base, predecessors, risk tier, model tier, and bounded contract;
- maintain per-task snapshot state: base, current, validated, reviewed, accepted HEADs;
- maintain a Git resource ledger for **all** workflow-created branches/worktrees, not only worktree-backed task branches;
- prevent hot-file overlap without an explicit plan;
- require committed, clean Developer handoffs with validation bound to exact SHAs;
- ensure authoritative Reviewer approval is bound to the exact final task HEAD;
- invalidate and repeat validation/review whenever delivered code changes;
- arbitrate Developer/Reviewer disputes without allowing arbitration to bypass re-review of changed code;
- use stronger review tiers for High/Critical risk or materially uncertain findings when required by `model-routing.md`;
- integrate only `ACCEPTED_HEAD` task snapshots into staging;
- integrate dependent tasks in topological order;
- keep the user's target branch unpolluted until the final certified result is ready;
- run final staging validation before Integration Review and require both to certify the same staging HEAD;
- detect user-target drift and reconcile it in staging, then rerun final gates if staging changes;
- promote only a certified staging/task snapshot that does not acquire new unreviewed tree changes during promotion;
- process every workflow-owned Git resource through mandatory cleanup and verify deletion or report a concrete retained-with-reason state;
- report what changed, exact validation/review/model-escalation status, residual risks, deferred findings, and retained temporary Git resources.

## Failure handling

If a worker fails, stalls, edits outside its task boundary, cannot validate its work, or discovers an invalid base/dependency assumption, narrow/reassign/replan rather than absorbing implementation into the Controller by default.

If an agent's reasoning repeatedly fails to converge and stronger routing is justified, escalate according to `model-routing.md` while preserving task state and evidence.

If parallel tasks develop unsafe overlap, stop further modification of that surface and serialize or redesign the dependency plan.

If the runtime refuses a new subagent because the agent/thread limit is reached, treat it as a scheduler event, not an implementation failure.

If a Bugfix cannot be reproduced, use alternative evidence without fabricating reproduction. If root cause remains unproven, label the result accurately as mitigation or hypothesis-driven work.

If a Reviewer finding causes a code change, the prior review approval is stale. Revalidate and re-review the new final HEAD.

If Integration Review causes a staging repair, both staging validation and review are stale for the new HEAD. Repeat the final gates.

If the user target moves during the workflow, reconcile the latest target into staging and rerun the final gates before promotion.

If cleanup cannot be proven safe, retain the workflow-owned resource and report the exact reason. Never force-delete uncertain or user-owned state.

## Completion criteria

Do not declare the work complete until all applicable conditions hold:

- user requirements and acceptance criteria are satisfied;
- project instructions were followed;
- no workflow work is mixed with unaccounted pre-existing user changes;
- task risk/model routing is recorded and material escalations are accounted for;
- every accepted task has `TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD`;
- for Bugfix, root cause/completion state and regression evidence are accurately documented;
- blocking review findings are resolved;
- High/Critical uncertainty was not left unresolved merely to avoid a stronger model tier;
- Complex task commits were integrated into workflow staging in dependency-safe order;
- Complex staging has `STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`;
- target drift, if any, was reconciled before promotion and the reconciled staging snapshot was re-certified;
- the certified result was promoted to the user target without introducing unreviewed semantic changes;
- no required review/repair activity was skipped because the agent pool was saturated;
- every workflow-created temporary branch/worktree—including Simple-task branches and the staging branch—was either safely deleted and verified absent or explicitly retained with a concrete reason reported to the user.
