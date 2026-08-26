# Commit-Bound Code State Machine

Use this reference primarily for **ORCHESTRATED** work, or when a STANDARD task explicitly needs strict commit-bound review/integration.

Do **not** require this state machine for FAST work. A lightweight STANDARD task also does not need it unless review/integration safety benefits from a stable commit snapshot.

## Why this exists

This state machine is for multi-agent and integration correctness: validation and review should certify the exact code that is integrated. It is deliberately heavier than FAST/STANDARD default execution.

## Repository preflight for ORCHESTRATED work

Before ORCHESTRATED implementation begins, the Controller records:

- `USER_TARGET_BRANCH`;
- `TARGET_BASE_COMMIT`;
- current `git status` / whether user work is dirty;
- existing worktrees;
- relevant existing local branches.

Preserve all pre-existing user state. Do not silently absorb unrelated uncommitted work.

## Per-task snapshot fields

For every ORCHESTRATED Developer task track:

- `TASK_BASE_COMMIT` — immutable task starting commit;
- `TASK_HEAD` — current committed task tip;
- `VALIDATED_HEAD` — exact commit on which required scoped validation passed;
- `REVIEWED_HEAD` — exact commit independently reviewed and approved;
- `ACCEPTED_HEAD` — exact commit accepted for integration.

When dependencies matter also track:

- `PREDECESSORS`;
- `BASE_REF`;
- `INTEGRATION_TARGET` — normally the workflow staging branch.

## Developer handoff gate

Before an ORCHESTRATED task enters authoritative review:

1. commit all intended task changes;
2. ensure the task worktree is clean;
3. record `TASK_HEAD = HEAD`;
4. run required scoped validation on that exact commit;
5. when it passes, record `VALIDATED_HEAD = TASK_HEAD`.

If `HEAD != VALIDATED_HEAD`, the previous validation is stale.

## Review gate

The Reviewer receives:

`TASK_BASE_COMMIT..TASK_HEAD`

If review passes:

`REVIEWED_HEAD = TASK_HEAD`

Approval applies only to that exact commit.

## Invalidation rule

Any delivered code/test/build/config change after validation or review creates a new task snapshot.

- `TASK_HEAD != VALIDATED_HEAD` -> rerun required validation;
- `TASK_HEAD != REVIEWED_HEAD` -> obtain independent review before ORCHESTRATED acceptance.

This includes fixes made in response to findings.

## Task acceptance invariant

Set `ACCEPTED_HEAD = TASK_HEAD` only when:

`TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD`

and the Controller has accepted the task outcome and dispositioned material findings.

Only `ACCEPTED_HEAD` enters staging.

## Complex staging state

Track:

- `STAGING_BRANCH`;
- `STAGING_BASE_COMMIT = TARGET_BASE_COMMIT`;
- `STAGING_HEAD`;
- `STAGING_VALIDATED_HEAD`;
- `STAGING_REVIEWED_HEAD`.

Integrate accepted tasks in dependency-safe order.

After intended tasks are integrated:

1. record `STAGING_HEAD`;
2. run final build/tests/integration/regression checks on that exact HEAD;
3. set `STAGING_VALIDATED_HEAD = STAGING_HEAD` when validation passes;
4. run Integration Review over `STAGING_BASE_COMMIT..STAGING_HEAD`;
5. set `STAGING_REVIEWED_HEAD = STAGING_HEAD` when review passes.

If staging changes afterward, stale certifications must be repeated until:

`STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`

## User-target drift

Before promotion, check whether the user target moved since `TARGET_BASE_COMMIT`.

If it moved:

- reconcile the latest target into staging, not directly on the user target;
- resolve conflicts in staging;
- update `STAGING_HEAD`;
- rerun final validation and Integration Review.

Only a validated+reviewed staging snapshot may be promoted.

## STANDARD optional use

A STANDARD task may use a simplified commit-bound review when it improves safety, for example a sensitive change that still has one owner.

In that case it is enough to record a stable base/head for the reviewed diff and ensure validation applies to the final reviewed state. Do not automatically introduce staging, worktrees, or the full ORCHESTRATED ledger.

## FAST exclusion

FAST completion does not require `TASK_HEAD`, `VALIDATED_HEAD`, `REVIEWED_HEAD`, or `ACCEPTED_HEAD` bookkeeping. FAST relies on final working-state targeted validation and diff inspection unless the user/project explicitly asks for stricter Git certification.