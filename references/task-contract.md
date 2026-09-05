# Task Brief and ORCHESTRATED Task Contract

Use different contract weight by workflow tier.

- **FAST**: no delegated task contract; the main session implements directly.
- **STANDARD**: use a concise boundary-ready task brief.
- **ORCHESTRATED**: use the full bounded task contract below.

Do not spend tokens filling fields that do not affect execution.

# Plan Readiness Gate

Before assigning a STANDARD or ORCHESTRATED Developer, ensure the task is sufficiently explicit for safe implementation.

Check five items:

1. **Goal** — what outcome must be produced;
2. **Scope** — what subsystem/files/behavior the Developer owns;
3. **Architectural decisions / hard constraints** — consequential choices, interfaces, compatibility rules, ownership rules, or invariants the Developer must not silently change;
4. **Non-goals** — nearby work that must not be changed;
5. **Validation** — how success will be verified.

If these are clear enough, mark the task `PLAN_READY = yes` and choose the Developer using `model-routing.md`.

Do **not** require the Controller to prescribe routine local tactics when repository evidence can resolve them safely. The Developer may choose existing helpers, local file structure, exact implementation sequence, and other reversible details inside the accepted boundary.

If a consequential item is unclear, do not ask the Developer to invent public architecture or contracts. The Controller should clarify/replan first, using repository exploration, Explorer, or Bug Investigator only when they add concrete value.

# STANDARD lightweight task brief

A STANDARD Developer normally needs only:

- **Goal**;
- **Scope**;
- **Architectural decisions / hard constraints** — only when material;
- **Out of scope / non-goals**;
- **Acceptance criteria**;
- **Validation**;
- **Risk note** — only material risk that changes validation/review behavior.

The Developer is expected to perform local code exploration inside this scope, verify the boundary against repository evidence, and choose routine implementation tactics without unnecessary Controller round-trips.

Do not require ORCHESTRATED Git fields, dependency graphs, model-audit fields, worktree data, or commit-state bookkeeping unless the STANDARD task specifically needs them.

A STANDARD handoff should normally contain:

1. what changed;
2. files/surfaces changed;
3. validation performed and result;
4. notable risk/limitation;
5. any consequential plan assumption that proved false or required Controller clarification;
6. for Bugfix, root cause and regression verification when material.

# ORCHESTRATED full task contract

For ORCHESTRATED work, read `code-state.md` and `model-routing.md` before assignment.

## Goal

State the user-visible or architectural outcome.

## Request type

Feature, Bugfix, Refactor, Performance, or Maintenance.

For a difficult Bugfix, include the current evidence/root-cause status from `bugfix.md`.

## Plan readiness

Record:

- `PLAN_READY` — yes/no;
- `ARCHITECTURAL_DECISIONS` — only consequential decisions or hard constraints the Developer must preserve;
- `NON_GOALS` — meaningful nearby work to avoid;
- `EXECUTION_AMBIGUITY` — Low / Medium / High when material;
- `TASK_HORIZON` — Short / Medium / Long when it affects routing;
- `INTEGRATION_BREADTH` — Local / Subsystem / Cross-system when it affects routing.

A Developer task should normally start only when `PLAN_READY = yes`.

Do not turn `ARCHITECTURAL_DECISIONS` into a line-by-line implementation recipe. Local tactics belong to the Developer unless changing them would alter architecture, public behavior, compatibility, persisted data, security boundaries, ownership/lifetime contracts, or other consequential behavior.

## Critical invariants

For High/Critical tasks, or other tasks where correctness depends on a small number of non-obvious rules, record a concise `CRITICAL_INVARIANTS` list.

Use this only for behavior that must remain true, such as:

- an operation is atomic on failure;
- rollback restores the exact prior state;
- a receipt/handle can be consumed only once;
- revision/version mismatch must reject the operation;
- persisted/protocol/public behavior remains backward compatible.

Normally keep this to **5-10 items or fewer**. Do not turn it into another design document.

The Reviewer uses these invariants to distinguish required defects from optional hardening.

## Role, risk, and model assignment

Risk and model selection serve different purposes:

- `RISK_LEVEL` — Low / Medium / High / Critical; primarily affects governance, validation, and review strength;
- `EXECUTION_AMBIGUITY` — remaining implementation reasoning difficulty;
- `TASK_HORIZON` — continuity requirement;
- `INTEGRATION_BREADTH` — interaction breadth;
- `COST_SENSITIVITY` — only when it materially affects routing;
- `ASSIGNED_MODEL`;
- `REASONING_EFFORT` — when the runtime/model exposes it and it matters;
- `MODEL_RATIONALE` — required when the routing choice is non-obvious;
- `ESCALATION_REASON` — populated when evidence justifies a stronger model or reasoning effort.

Do not upgrade the Developer only because `RISK_LEVEL` is High. A high-risk but explicit implementation can remain on a cost-efficient model with stronger validation/review.

Conversely, long-horizon or Cross-system work may justify GPT-6 Astra even when the change is not Critical.

## Scope

Define the module/file/interface ownership boundary.

## Out of scope

List meaningful nearby refactors, API redesigns, dependency upgrades, or other changes that require Controller approval.

## Relevant orchestration context

Provide only what this Developer needs from Explorer/Bug Investigator output: dependencies, critical-path information, hot-file warnings, shared APIs, known risks, established root-cause evidence, and consequential decisions.

Do not paste a large Explorer report when the Developer can read the implementation surface directly or retained/searchable context already contains the evidence.

## Git base and dependencies

Record:

- `BASE_REF`;
- `TASK_BASE_COMMIT`;
- `PREDECESSORS`;
- `INTEGRATION_TARGET` — normally staging.

A dependent task must not start until required predecessor outputs are accepted and present in its base.

## Acceptance criteria

Use observable conditions covering behavior, compatibility, failure handling, and performance when material.

For Bugfix include the symptom/root-cause state and regression expectation.

## Validation plan

Use the Validation Pyramid instead of repeatedly running the broadest available suite.

Record only what this task needs:

- **Inner-loop checks** — fastest affected-target compile and directly relevant tests/checks used while implementing or repairing;
- **Task-gate validation** — the task-specific suite that must pass on the final committed `TASK_HEAD` before authoritative review/acceptance;
- **Staging-owned validation** — expensive full-suite, integration, stress, real-data, or whole-project checks that should normally wait until staging unless this task specifically requires them earlier.

Once required targeted checks pass, do not broaden or repeat validation unless later changes, failures, or concrete unresolved risk invalidate that evidence.

Do not require every Developer to rerun expensive staging-owned validation independently.

## Change permissions

State permissions where ambiguity is risky, such as:

- public API changes;
- dependencies;
- build configuration;
- schema/protocol/serialization;
- cross-module refactors;
- diagnostic instrumentation.

The default expectation is the **smallest implementation that satisfies the stated boundary, Critical Invariants, and acceptance criteria**.

# Developer blocked-boundary handoff

If the Developer discovers that a consequential assumption is false, it should not silently redesign the affected boundary.

First identify whether other work can continue safely.

Return:

- **Expected** — what the task contract assumed;
- **Actual** — what the repository shows;
- **Impact** — why the affected slice cannot follow the current boundary safely;
- **Blocked slice** — work that must wait for a decision;
- **Independent work** — reversible work that can safely continue without prejudging the decision, if any;
- **Decision needed** — the smallest Controller decision/replan required.

If the runtime supports asking for a consequential decision while continuing independent work, prefer that over idling the entire task.

Do not invoke this handoff for routine local implementation details that project conventions can resolve safely.

# Context continuity

For long-running tasks, preserve useful role-owned context when it reduces rediscovery.

A Developer should normally keep the same context across implementation and compatible repair work. A Controller may keep long-lived orchestration state. Split contexts when parallel ownership, independent review, Git isolation, or a fresh perspective provides concrete value.

Do not use context continuity to weaken read-only Reviewer boundaries or mix unrelated task ownership.

# Repair batching

When a Reviewer returns multiple compatible findings, treat them as one repair batch when practical.

The Developer should:

1. disposition the full finding set;
2. fix all Confirmed required defects/approved contract gaps together when safe;
3. add/update the related regression tests together;
4. run the relevant inner-loop checks during repair;
5. create a meaningful repair commit/snapshot rather than one commit per tiny finding when repository policy permits;
6. run the Task-gate validation once on the final repaired `TASK_HEAD` before re-review.

Prefer returning the repair batch to the same Developer context when that context still owns the task and retains useful state. Create a fresh repair agent only when availability, independence, isolation, or lost context justifies it.

Do not implement Optional Hardening or Deferred findings unless the Controller/user explicitly accepts the scope expansion.

# ORCHESTRATED Developer handoff

Before handoff:

1. all intended changes are committed;
2. task worktree is clean;
3. `TASK_HEAD = HEAD` is recorded;
4. required **Task-gate validation** passes on that exact commit;
5. `VALIDATED_HEAD = TASK_HEAD` is recorded.

Return:

- implementation summary;
- changed/added files;
- any necessary local implementation decision that affects later integration understanding;
- Critical Invariant status when applicable;
- `TASK_BASE_COMMIT` and `TASK_HEAD`;
- clean worktree confirmation;
- validation commands/results and `VALIDATED_HEAD`;
- limitations/residual risks;
- contract deviations or corrected consequential assumptions;
- branch/worktree state.

For difficult Bugfix work also return Symptom, Root Cause, Evidence, Fix, Regression Verification, Residual Risk, and precise completion state.

The Controller rejects an ORCHESTRATED handoff whose intended code is uncommitted, whose worktree is dirty, or whose Task-gate validation does not certify the final reported `TASK_HEAD`.
