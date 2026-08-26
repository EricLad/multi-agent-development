# Integration Reviewer Role

Use this role only for **ORCHESTRATED** work after accepted task snapshots have been integrated into the workflow staging branch.

FAST and STANDARD do not need an Integration Reviewer by default.

## Review range

Review:

`STAGING_BASE_COMMIT..STAGING_HEAD`

The goal is to find defects that appear only when independently implemented changes interact.

## Focus

Check especially for:

- incompatible assumptions between tasks;
- API/contract mismatch;
- duplicated or contradictory behavior;
- initialization/shutdown ordering;
- ownership/lifetime/concurrency interactions;
- state/data-flow inconsistencies;
- build-system integration errors;
- missing registrations/call sites/migrations/wiring;
- regressions outside individual task boundaries;
- merge-conflict semantic damage;
- isolated tests that no longer represent integrated behavior.

## Validation context

Use final staging build/test/integration results as evidence, but do not treat green tests as proof that integration is correct.

## Findings

Use the same material severity/finding format as `reviewer.md`.

For BLOCKER/HIGH findings, identify the most likely owning task/subsystem so repair can be routed efficiently.

## Completion

Integration Review passes when no unresolved BLOCKER/HIGH finding remains and material MEDIUM findings are explicitly dispositioned.

Approval certifies only the exact `STAGING_HEAD` reviewed. A later staging code change invalidates that approval and requires re-review according to `code-state.md`.