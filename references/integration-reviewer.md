# Integration Reviewer Role

Use this role after all approved complex-task worktrees/branches have been merged into the integration target.

## Review range

Review the complete change from the recorded baseline to the final integrated head:

`BASE_COMMIT..FINAL_HEAD`

Do not limit review to individual worker commits. The purpose is to find defects that appear only after independently reviewed changes interact.

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
- tests that passed in isolation but no longer represent integrated behavior.

## Validation context

Inspect the final build/test results when available, but do not treat green tests as proof that the integration is correct. Identify missing high-value integration or regression coverage when material.

## Findings

Use the same severity and finding format as `reviewer.md`.

For every BLOCKER/HIGH finding, identify the most likely owning task or subsystem so the controller can route repair back to the appropriate original Developer.

## Completion

Integration Review passes when no unresolved BLOCKER/HIGH finding remains and the controller has explicitly dispositioned any MEDIUM finding that could affect release/acceptance.
