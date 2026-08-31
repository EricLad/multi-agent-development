# Role + Execution-Ambiguity Model Routing

Apply model routing only when subagents are actually used. Do not create subagents or bookkeeping solely to exercise this policy.

The central rule is:

> **Risk controls governance; execution ambiguity controls Developer model escalation.**

A high-risk task does not automatically need a stronger Developer when the implementation plan is already explicit. Instead, high risk should normally strengthen validation and review. Upgrade the Developer only when the implementation itself still requires materially harder reasoning.

# Workflow-tier interaction

## FAST

No delegated-role routing by default. Use the current main-session model and keep the task in one coding context.

## STANDARD

Use one Developer when delegation is useful.

Default:

- **Developer -> GPT-5.6 Terra medium**

Use an independent Reviewer only when risk, uncertainty, weak tests, or public behavior justifies it.

## ORCHESTRATED

After the Controller makes each task implementation-ready, use:

- **Developer -> GPT-5.6 Terra medium by default**
- stronger Developer models only when execution ambiguity remains or new evidence invalidates the plan

The purpose of decomposition is to turn complex work into clear bounded implementation tasks that Terra medium can execute efficiently.

# Plan Readiness before Developer assignment

Do not compensate for an unclear task by immediately selecting a stronger Developer model.

Before starting a STANDARD or ORCHESTRATED Developer, the Controller should ensure the plan is sufficiently explicit:

1. **Goal** — the required outcome is clear;
2. **Scope** — ownership/files/subsystem boundary is clear enough;
3. **Implementation approach** — the intended solution and important constraints are stated;
4. **Non-goals** — nearby work that must not be changed is clear;
5. **Validation** — success can be checked with concrete build/test/behavior verification.

When these are satisfied, treat the task as `PLAN_READY` and prefer Terra medium.

If they are not satisfied, improve the plan first using the Controller, repository exploration, or Bug Investigator/Explorer when genuinely needed.

# Developer routing

## Default: Terra medium

Use **GPT-5.6 Terra medium** for normal Developer work when the task is implementation-ready, including:

- ordinary production features;
- bounded multi-file changes;
- ORCHESTRATED subtasks with clear ownership and interfaces;
- high-risk changes whose design/algorithm/compatibility strategy has already been decided and whose implementation is concrete;
- review-finding repairs when the required correction is clear.

The Developer is an **implementation agent**, not a second architect. It should verify the plan against the local code, implement narrowly, validate, and avoid inventing additional abstractions or unrelated refactors.

## Escalate to Terra xhigh

Upgrade the Developer to **Terra xhigh** when implementation remains materially ambiguous after reasonable local inspection, for example:

- the task contract conflicts with the actual architecture;
- multiple implementation strategies remain and choosing among them materially affects architecture or compatibility;
- a required API/ownership/lifecycle assumption is false;
- repeated build/test failures do not converge to a localized cause;
- unexpected cross-module coupling appears and cannot be resolved by a small plan correction;
- the Developer would otherwise need to redesign the task rather than implement it.

Prefer returning the problem to the Controller for replanning when possible. Escalation is appropriate when the unresolved reasoning belongs inside the implementation task itself.

## Escalate to Terra max / Sol

Use **Terra max** only when the implementation itself requires unusually difficult reasoning such as unresolved concurrency/lifetime/state-machine interactions, subtle memory/resource safety, or similarly difficult semantics.

Use **Sol xhigh/max** only for Critical or repeatedly unresolved implementation problems where lower tiers have failed to converge.

Do not escalate merely because the task is high risk, long, or spans many lines.

# Other role routing

## Explorer

Default: **Luna max**.

Escalate to Terra xhigh only when the orchestration map itself needs difficult architectural/ownership inference.

Explorer remains optional and should exist only when decomposition/orchestration needs a global map.

## Bug Investigator

- straightforward diagnosis -> Luna max;
- non-trivial cross-module/root-cause work -> Terra xhigh;
- concurrency/lifetime/state corruption or stubborn multi-hypothesis diagnosis -> Terra max;
- Critical/repeatedly unresolved -> Sol xhigh/max.

The Investigator should resolve uncertainty before implementation when possible so the resulting Developer task can return to Terra medium.

## Reviewer

Default: **Luna max**.

Escalate to **Terra xhigh** for High/Critical risk, material uncertainty, security/concurrency/lifetime/schema/protocol/broad public behavior, or difficult evidence-based disputes.

A Terra medium Developer does not automatically require a stronger Reviewer. Review tier is determined by the risk and difficulty of finding defects, not by the Developer's model name.

## Integration Reviewer

Default: **Terra xhigh**.

Escalate to Terra max for High-risk integrated changes, meaningful conflict resolution, broad coupling, concurrency/security/data-migration concerns. Sol is reserved for Critical/unresolved integration uncertainty.

# Developer blocked-plan rule

When a Developer discovers that a key plan assumption is wrong, it should not silently redesign the solution or expand scope.

Return a concise block report containing:

- expected assumption;
- actual repository evidence;
- why the current plan cannot be followed safely;
- smallest decision or plan correction needed.

The Controller then replans, after which the same Terra medium Developer tier may often continue.

# Cost and quality control

Prefer this sequence:

`make plan explicit -> Terra medium implements -> validate -> review proportional to risk`

not:

`unclear plan -> stronger Developer guesses -> larger diff -> more review`

Use stronger models for unresolved reasoning, not as a substitute for task clarity.

# Audit fields

Keep bookkeeping proportional to the workflow. For ORCHESTRATED work, record when useful:

- `RISK_LEVEL` — for governance/review decisions;
- `PLAN_READY` — yes/no;
- `EXECUTION_AMBIGUITY` — Low / Medium / High when material;
- `ASSIGNED_MODEL`;
- `MODEL_RATIONALE` when not using Terra medium for a Developer;
- `ESCALATION_REASON` when upgraded.

These fields are not completion ceremony for FAST/STANDARD.