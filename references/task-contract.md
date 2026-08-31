# Task Brief and ORCHESTRATED Task Contract

Use different contract weight by workflow tier.

- **FAST**: no delegated task contract; the main session implements directly.
- **STANDARD**: use a concise implementation-ready task brief.
- **ORCHESTRATED**: use the full bounded task contract below.

Do not spend tokens filling fields that do not affect execution.

# Plan Readiness Gate

Before assigning a STANDARD or ORCHESTRATED Developer, ensure the task is sufficiently explicit for implementation.

Check five items:

1. **Goal** — what outcome must be produced;
2. **Scope** — what subsystem/files/behavior the Developer owns;
3. **Implementation approach** — the intended solution, interfaces, sequence, or important constraints;
4. **Non-goals** — nearby work that must not be changed;
5. **Validation** — how success will be verified.

If these are clear enough, mark the task `PLAN_READY = yes` and prefer **GPT-5.6 Terra medium** for the Developer.

If a key item is unclear, do not ask the Developer to invent the architecture. The Controller should clarify/replan first, using repository exploration, Explorer, or Bug Investigator only when they add concrete value.

# STANDARD lightweight task brief

A STANDARD Developer normally needs only:

- **Goal**;
- **Scope**;
- **Implementation approach**;
- **Out of scope / non-goals**;
- **Acceptance criteria**;
- **Validation**;
- **Risk note** — only material risk that changes validation/review behavior.

The Developer is expected to perform their own local code exploration inside this scope and verify that the plan matches the repository before editing.

Do not require ORCHESTRATED Git fields, dependency graphs, model-audit fields, worktree data, or commit-state bookkeeping unless the STANDARD task specifically needs them.

A STANDARD handoff should normally contain:

1. what changed;
2. files/surfaces changed;
3. validation performed and result;
4. notable risk/limitation;
5. any plan assumption that proved false or required Controller clarification;
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
- `IMPLEMENTATION_APPROACH` — concise intended solution;
- `NON_GOALS` — meaningful nearby work to avoid;
- `EXECUTION_AMBIGUITY` — Low / Medium / High when material.

A Developer task should normally start only when `PLAN_READY = yes`.

## Role, risk, and model assignment

Risk and model selection serve different purposes:

- `RISK_LEVEL` — Low / Medium / High / Critical; primarily affects governance, validation, and review strength;
- `ASSIGNED_MODEL` — Developer default is **GPT-5.6 Terra medium** when the task is implementation-ready;
- `MODEL_RATIONALE` — required when the Developer is upgraded above Terra medium;
- `ESCALATION_REASON` — populated when unresolved execution ambiguity requires a stronger tier.

Do not upgrade the Developer only because `RISK_LEVEL` is High. A high-risk but explicit implementation can remain on Terra medium with stronger validation/review.

## Scope

Define the module/file/interface ownership boundary.

## Out of scope

List meaningful nearby refactors, API redesigns, dependency upgrades, or other changes that require Controller approval.

## Relevant orchestration context

Provide only what this Developer needs from Explorer/Bug Investigator output: dependencies, hot-file warnings, shared APIs, known risks, or established root-cause evidence.

Do not paste a large Explorer report when the Developer can read the implementation surface directly.

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

## Validation

Specify the minimum relevant build target, tests, regression/stress checks, static/lint checks, or manual verification.

For ORCHESTRATED work, validation applies to the final committed `TASK_HEAD`.

## Change permissions

State permissions where ambiguity is risky, such as:

- public API changes;
- dependencies;
- build configuration;
- schema/protocol/serialization;
- cross-module refactors;
- diagnostic instrumentation.

The default expectation is the **smallest implementation that satisfies the stated approach and acceptance criteria**.

# Developer blocked-plan handoff

If the Developer discovers that a key plan assumption is false, it should stop before broad redesign and return:

- **Expected** — what the task contract assumed;
- **Actual** — what the repository shows;
- **Impact** — why the current plan cannot be followed safely;
- **Decision needed** — the smallest Controller decision/replan required.

After the Controller corrects the plan, the Developer can usually continue on Terra medium.

# ORCHESTRATED Developer handoff

Before handoff:

1. all intended changes are committed;
2. task worktree is clean;
3. `TASK_HEAD = HEAD` is recorded;
4. required validation passes on that exact commit;
5. `VALIDATED_HEAD = TASK_HEAD` is recorded.

Return:

- implementation summary;
- changed/added files;
- any necessary low-level implementation decision not already fixed by the contract;
- `TASK_BASE_COMMIT` and `TASK_HEAD`;
- clean worktree confirmation;
- validation commands/results and `VALIDATED_HEAD`;
- limitations/residual risks;
- contract deviations or corrected plan assumptions;
- branch/worktree state.

For difficult Bugfix work also return Symptom, Root Cause, Evidence, Fix, Regression Verification, Residual Risk, and precise completion state.

The Controller rejects an ORCHESTRATED handoff whose intended code is uncommitted, whose worktree is dirty, or whose validation does not certify the final reported `TASK_HEAD`.