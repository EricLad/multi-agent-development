# Integration Reviewer Role

Use this role only after all intended Controller-accepted Complex-task work has been integrated into the workflow-owned staging branch and the initial final validation suite has passed on the exact current staging HEAD.

Read `code-state.md` before review.

## Review target

The Controller must provide:

- `STAGING_BASE_COMMIT`;
- `STAGING_HEAD`;
- final validation evidence bound to that same `STAGING_HEAD`.

Review the complete integrated change:

`STAGING_BASE_COMMIT..STAGING_HEAD`

Do not limit review to individual worker commits. The purpose is to find defects that appear only after independently reviewed changes interact.

Do not review an unspecified dirty working-tree state. The staging branch should be committed and clean.

## Snapshot certification

An Integration Review approval certifies only the exact `STAGING_HEAD` that was reviewed.

When the review passes, report the approved commit explicitly so the Controller can record:

`STAGING_REVIEWED_HEAD = STAGING_HEAD`

If any repair, target-drift integration, conflict resolution, build/config update, or other delivered change modifies staging afterward, the previous Integration Review is stale. Final validation and Integration Review must be repeated on the new staging HEAD until the same exact commit is both validated and reviewed.

## Validation context

The initial clean/full/integration/regression validation should run **before** this Integration Review so the Reviewer can inspect its results as evidence.

Green tests are not proof that integration is correct. Identify missing high-value integration or regression coverage when material.

The final readiness invariant is:

`STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`

## Focus areas

Check especially for:

- incompatible assumptions between tasks;
- public/private API mismatch;
- duplicated or contradictory implementation;
- initialization/shutdown ordering problems;
- ownership and lifetime interactions;
- concurrency and synchronization interactions;
- state/data-flow inconsistencies;
- build-system integration errors;
- missing registrations, call sites, migrations, or wiring;
- behavior regressions outside individual task boundaries;
- merge-conflict resolutions that changed semantics;
- tests that passed in isolation but no longer represent integrated behavior;
- dependency-order mistakes or task bases that omitted required predecessor behavior;
- staging changes that were never independently reviewed at task level and are not justified as integration-only resolutions.

## User-target drift

If the user target branch moved after staging was originally based, final target reconciliation must happen in staging, not directly on the user target after this review.

If the Controller incorporates the newer target state into staging, the staging HEAD changes and this review becomes stale. Re-run final validation and Integration Review on the reconciled staging HEAD.

## Findings

Use the same severity and finding format as `reviewer.md`.

For every BLOCKER/HIGH finding, identify the most likely owning task or subsystem so the Controller can route repair back to the appropriate original Developer.

A repair that changes staging invalidates the current Integration Review approval.

## Completion

Integration Review passes only when:

- no unresolved BLOCKER/HIGH finding remains;
- the Controller has explicitly dispositioned any MEDIUM finding that could affect release/acceptance;
- the exact approved `STAGING_HEAD` is stated;
- the same staging HEAD already has valid final validation evidence.

Only then may the Controller consider that staging snapshot eligible for promotion to the user's target branch.