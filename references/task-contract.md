# Task Brief and ORCHESTRATED Task Contract

Use different contract weight by workflow tier.

- **FAST**: no delegated task contract; the main session implements directly.
- **STANDARD**: use a concise task brief.
- **ORCHESTRATED**: use the full bounded task contract below.

Do not spend tokens filling fields that do not affect execution.

# STANDARD lightweight task brief

A STANDARD Developer normally needs only:

- **Goal** — what outcome to produce;
- **Scope** — subsystem/files or behavior to own;
- **Out of scope** — only important nearby work to avoid;
- **Acceptance criteria** — observable success conditions;
- **Validation** — relevant build/test/check commands or expected verification;
- **Risk note** — only material risk that changes implementation/review behavior.

The Developer is expected to perform their own code exploration inside this scope.

Do not require ORCHESTRATED Git fields, dependency graphs, model-audit fields, worktree data, or commit-state bookkeeping unless the STANDARD task specifically needs them.

A STANDARD handoff should normally contain:

1. what changed;
2. files/surfaces changed;
3. validation performed and result;
4. notable risk/limitation;
5. for Bugfix, root cause and regression verification when material.

# ORCHESTRATED full task contract

For ORCHESTRATED work, read `code-state.md` and `model-routing.md` before assignment.

## Goal

State the user-visible or architectural outcome.

## Request type

Feature, Bugfix, Refactor, Performance, or Maintenance.

For a difficult Bugfix, include the current evidence/root-cause status from `bugfix.md`.

## Role, risk, and model assignment

Record when useful:

- `ROLE`;
- `RISK_LEVEL` — Low / Medium / High / Critical;
- `ASSIGNED_MODEL`;
- `REASONING_EFFORT`;
- `MODEL_RATIONALE` when deviating from the role default;
- `ESCALATION_REASON` if upgraded during execution.

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

Explicitly state permissions only where ambiguity is risky, such as:

- public API changes;
- dependencies;
- build configuration;
- schema/protocol/serialization;
- cross-module refactors;
- diagnostic instrumentation.

## ORCHESTRATED Developer handoff

Before handoff:

1. all intended changes are committed;
2. task worktree is clean;
3. `TASK_HEAD = HEAD` is recorded;
4. required validation passes on that exact commit;
5. `VALIDATED_HEAD = TASK_HEAD` is recorded.

Return:

- implementation summary;
- changed/added files;
- important design decisions;
- `TASK_BASE_COMMIT` and `TASK_HEAD`;
- clean worktree confirmation;
- validation commands/results and `VALIDATED_HEAD`;
- limitations/residual risks;
- contract deviations;
- branch/worktree state.

For difficult Bugfix work also return Symptom, Root Cause, Evidence, Fix, Regression Verification, Residual Risk, and precise completion state.

The Controller rejects an ORCHESTRATED handoff whose intended code is uncommitted, whose worktree is dirty, or whose validation does not certify the final reported `TASK_HEAD`.