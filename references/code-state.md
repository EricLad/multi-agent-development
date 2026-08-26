# Commit-Bound Code State Machine

Use this reference for every implementation task. Quality gates certify an exact Git snapshot, not an abstract task.

## Repository preflight

Before any Developer edits code, the Controller records:

- `USER_TARGET_BRANCH` — the user branch that should ultimately receive the work;
- `TARGET_BASE_COMMIT` — its HEAD when the workflow begins;
- current `git status` / whether the user's working tree is dirty;
- existing worktrees;
- existing local branches relevant to names the workflow may create.

Do not mix workflow changes with pre-existing uncommitted user changes. If the current working tree is dirty, preserve it. Prefer an isolated workflow worktree/branch from a committed ref. If the requested change depends on the user's uncommitted changes, treat that as a material dependency requiring an explicit safe plan rather than silently absorbing them.

## Per-task snapshot fields

For every Developer task track:

- `TASK_BASE_COMMIT` — immutable commit the task branch/worktree started from;
- `TASK_HEAD` — current committed task tip;
- `VALIDATED_HEAD` — exact commit on which required scoped build/tests passed;
- `REVIEWED_HEAD` — exact commit independently reviewed and approved;
- `ACCEPTED_HEAD` — exact commit accepted by the Controller for integration.

For dependency-aware work also track:

- `PREDECESSORS` — task IDs/commits that must be present before the task starts;
- `BASE_REF` — branch/ref used to create the task branch;
- `INTEGRATION_TARGET` — for Complex work this should normally be the workflow staging branch, not the user's target branch.

## Developer handoff gate

Before a task is eligible for authoritative review:

1. all intended task changes are committed;
2. the task working tree is clean;
3. record `TASK_HEAD = HEAD`;
4. run the required scoped validation against that exact committed HEAD;
5. only after it passes, record `VALIDATED_HEAD = TASK_HEAD`.

Do not accept validation performed on an older snapshot. If `HEAD != VALIDATED_HEAD`, the validation result is stale.

## Review gate

The authoritative Reviewer receives an explicit range:

`TASK_BASE_COMMIT..TASK_HEAD`

and the task contract. If the review passes, record:

`REVIEWED_HEAD = TASK_HEAD`

A review approval applies only to that exact commit.

## Invalidation rule

Any code/test/build/config change after validation or review creates a new `TASK_HEAD` and invalidates stale certifications:

- if `TASK_HEAD != VALIDATED_HEAD`, rerun the required scoped validation;
- if `TASK_HEAD != REVIEWED_HEAD`, obtain a new independent review before acceptance.

This includes fixes made in response to Reviewer findings. A BLOCKER/HIGH repair must never be merged based only on the review of the pre-fix commit.

The same Reviewer may perform a re-review when still independent from the Developer, but the approval must name the new exact HEAD. The Controller may request a focused incremental inspection for a tiny follow-up, but authoritative approval still attaches to the final task HEAD and must consider interaction with the previously reviewed task diff.

## Task acceptance invariant

Set `ACCEPTED_HEAD = TASK_HEAD` only when all applicable conditions hold:

`TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD`

and the Controller has accepted the task contract outcome and dispositioned material findings.

Only `ACCEPTED_HEAD` may be integrated.

## Simple-task promotion and target drift

For a Simple task, prefer a workflow-owned temporary task branch even when no additional worktree is needed. Keep `USER_TARGET_BRANCH` unchanged while the task is implemented, validated, and reviewed.

Before promotion, compare the current user-target HEAD with `TARGET_BASE_COMMIT`.

If the user target has not moved, promote the exact `ACCEPTED_HEAD` without introducing additional tree changes.

If the user target moved:

1. do not resolve final conflicts directly on the user target;
2. incorporate the latest user-target state into the temporary task branch using the repository-appropriate merge/rebase strategy;
3. resolve any conflicts on the temporary branch;
4. the resulting commit becomes a new `TASK_HEAD`;
5. prior `VALIDATED_HEAD`, `REVIEWED_HEAD`, and `ACCEPTED_HEAD` are stale unless they equal that new HEAD;
6. rerun scoped validation and independent review;
7. set a new `ACCEPTED_HEAD` only when the task invariant is restored;
8. then promote the certified temporary-branch result to the user target.

This keeps conflict resolution inside a branch that can still be validated/reviewed before the user's target moves.

## Complex staging state

For Complex work, create a workflow-owned staging/integration branch from `TARGET_BASE_COMMIT`. Track:

- `STAGING_BRANCH`;
- `STAGING_BASE_COMMIT = TARGET_BASE_COMMIT`;
- `STAGING_HEAD`;
- `STAGING_VALIDATED_HEAD`;
- `STAGING_REVIEWED_HEAD`.

Integrate accepted task branches into staging in dependency/topological order. Do not merge unaccepted task tips.

After all intended task work is integrated:

1. record `STAGING_HEAD`;
2. run the clean/full/integration/regression validation suite on that exact HEAD;
3. if it passes, set `STAGING_VALIDATED_HEAD = STAGING_HEAD`;
4. run Integration Review over `STAGING_BASE_COMMIT..STAGING_HEAD`, with validation results available as context;
5. if it passes, set `STAGING_REVIEWED_HEAD = STAGING_HEAD`.

If any repair changes staging, both prior staging validation and Integration Review are stale. Repeat the affected repair cycle until:

`STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`

## Complex user-target drift gate

Before integrating staging into `USER_TARGET_BRANCH`, verify whether the user target still points to `TARGET_BASE_COMMIT`.

If it moved during the workflow:

- do not resolve final conflicts directly on the user target and then declare success;
- first incorporate the latest user-target state into the workflow staging branch using the repository-appropriate merge/rebase strategy;
- resolve conflicts in staging;
- update `STAGING_HEAD`;
- rerun final validation and Integration Review on the new staging HEAD.

Only a staging snapshot that is both validated and reviewed may be promoted to the user target.

## Promotion gate

Prefer a promotion that does not change the already certified tree, for example a fast-forward when feasible or a normal merge without semantic conflict resolution.

If promotion itself requires code conflict resolution or otherwise changes the resulting tree, the promoted result has not been certified. Move that resolution back into the temporary task branch (Simple) or staging branch (Complex) and repeat the applicable final gates before updating the user target.

## Final invariant

A task or staging branch is not "good" because an agent says it is done. It is good only when the exact commit being integrated has the required validation and independent-review certifications.