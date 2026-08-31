# Quality Gates and Git / Worktree Safety

Use validation and Git controls in proportion to the workflow tier.

# FAST

FAST should stay lightweight.

Before editing:

- inspect repository/project instructions;
- inspect current Git status enough to avoid overwriting or confusing unrelated user changes.

During completion:

- run the most relevant targeted build/test/checks;
- inspect the final diff/state;
- do not create a workflow branch/worktree/staging branch merely for ceremony;
- do not require commit/SHA certification unless the user/project explicitly asks for it.

# STANDARD

STANDARD normally uses one Developer and targeted validation.

Recommended:

- preserve user changes;
- use the repository's normal branch/commit practice;
- run relevant build/tests/checks after the final implementation state;
- when an independent Reviewer is justified, give the Reviewer a stable final diff/state; a commit boundary is useful but not mandatory unless needed for safety/auditability;
- group compatible review repairs instead of creating one test/review cycle per tiny finding.

Do not create Agent Pool, worktrees, staging, resource ledgers, or full commit-state bookkeeping unless the task upgrades to ORCHESTRATED.

# ORCHESTRATED

The rest of this reference applies to ORCHESTRATED work.

Read `code-state.md`.

## Git preflight

Record:

- user target branch and starting HEAD;
- working-tree dirty state;
- existing worktrees;
- relevant existing local branches.

Preserve pre-existing user work.

# Validation Pyramid

Use the **narrowest validation that answers the current question**, then broaden only at quality gates.

## Level 1 — Inner loop

During implementation or repair:

- compile the affected target/module when practical;
- run directly relevant focused tests/checks;
- use deterministic repro checks for the specific bug/behavior;
- do not repeatedly run expensive full suites or real-data/system tests after every small edit unless the failure mode requires them.

Level 1 is for fast feedback. It does not certify an ORCHESTRATED task for review.

## Level 2 — Task gate

Before authoritative per-task review/acceptance:

1. batch intended implementation/repair changes into a meaningful final task snapshot;
2. commit all intended task changes;
3. ensure the task worktree is clean;
4. record `TASK_HEAD`;
5. run the task-specific build/tests/checks required by the contract on that exact HEAD;
6. set `VALIDATED_HEAD = TASK_HEAD` when they pass.

Do not automatically include expensive whole-project, real-data, broad integration, or stress suites in every Task gate when those checks are explicitly owned by staging.

## Level 3 — Staging gate

After accepted tasks are integrated into staging, run the expensive broad checks on the integrated snapshot:

- clean/fresh build when appropriate;
- project-appropriate full suite;
- integration/regression checks;
- stress/performance checks when required;
- real-data/real-environment checks required by project policy.

The default goal is to run these broad checks **once per stable staging candidate**, not once per Developer task.

If staging changes after a material repair, rerun only the levels invalidated by that change, while always satisfying project-specific mandatory final checks.

## Validation ownership

The Task Contract should identify which checks belong to:

- inner loop;
- Task gate;
- staging gate.

This prevents multiple Developers from independently repeating the same expensive integration validation.

# Commit batching

Commit boundaries exist to identify meaningful reviewable snapshots, not to create ceremony.

For one ORCHESTRATED task, prefer when repository/user policy permits:

- one meaningful implementation snapshot; and
- at most one grouped review-repair snapshot for compatible findings.

Additional commits are fine when they represent real independent units, but avoid one commit per tiny finding, assertion, or test case solely because review found them sequentially.

Every authoritative review still applies to the final exact `TASK_HEAD`.

# Per-task review/acceptance gate

A task is ready for authoritative review only when Level 2 Task-gate validation certifies the exact final `TASK_HEAD`.

A later delivered-content change makes stale validation/review invalid according to `code-state.md`.

When review returns multiple compatible Required Defects, fix them as one repair batch when safe, then perform one Level 2 validation on the resulting final repaired HEAD before re-review.

## Staging branch

For ORCHESTRATED work:

- record `TARGET_BASE_COMMIT`;
- create a workflow-owned staging branch from it;
- integrate only Controller-accepted task snapshots;
- keep the user target unchanged until the staging result passes final gates.

## Git resource ledger

Track every workflow-created Git resource:

- task/resource ID;
- local branch;
- worktree path if any;
- role: task branch, staging branch, or other workflow-owned temporary resource;
- base ref/commit;
- predecessor/dependency information;
- integration target;
- lifecycle state.

Do not infer ownership later from branch names alone.

## Worktree rules

- use worktrees only for genuinely parallel/isolated implementation;
- one branch/worktree per independent task;
- do not let multiple Developers unknowingly own the same hot file/shared API;
- do not delete dirty worktrees;
- never reset/delete pre-existing user resources.

Create each task from the dependency-correct base and record that base.

## Dependency-aware task creation

A task becomes ready only when required predecessors are accepted and the selected base contains those outputs.

Do not silently change a task base after implementation begins without invalidating/re-running affected gates.

Integrate accepted tasks in dependency/topological order.

## Per-task staging gate

Before staging integration verify:

- acceptance criteria and Critical Invariants are met;
- `TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD`;
- worktree clean;
- BLOCKER/HIGH Required Defects resolved;
- material MEDIUM findings and Contract Gaps dispositioned;
- Optional Hardening/Deferred work has not silently expanded current scope.

Merge only `ACCEPTED_HEAD`.

## Integration strategy

Prefer normal merge-based integration for workflow task branches when repository policy allows because provenance/cleanup are easier to prove.

If squash/rebase/cherry-pick is required, record the integration method and evidence mapping the accepted change into staging.

## Final ORCHESTRATED gate

After accepted tasks are in staging:

1. record `STAGING_HEAD`;
2. perform Level 3 Staging-gate validation;
3. set `STAGING_VALIDATED_HEAD = STAGING_HEAD` when validation passes;
4. run Integration Review over `STAGING_BASE_COMMIT..STAGING_HEAD`;
5. batch compatible integration repairs when needed;
6. after repair, rerun affected validation and the mandatory final staging checks, then re-review the final `STAGING_HEAD`;
7. set `STAGING_REVIEWED_HEAD = STAGING_HEAD` when review passes;
8. if substantial new material findings continue after a second repair/re-review cycle, invoke the Convergence Gate rather than looping indefinitely;
9. reconcile user-target drift in staging if needed and re-run invalidated final gates;
10. promote only the certified staging result.

Readiness invariant:

`STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`

## Promotion

Promotion must not introduce unreviewed semantic changes. Resolve conflicts in staging, revalidate/re-review, then promote.

# Mandatory ORCHESTRATED cleanup

Process every workflow-owned temporary resource after final acceptance.

## Ownership

Auto-delete only resources proven to be created by this workflow. Never delete user-owned/uncertain resources or remote branches merely because names match.

## Worktree cleanliness

For worktrees, verify clean state before removal:

```text
git -C <worktree-path> status --porcelain
```

Do not force-remove dirty worktrees.

## Integration proof

For merge-based integration, ancestry proof may use:

```text
git merge-base --is-ancestor <temporary-branch> <integration-target>
```

For non-ancestry integration require recorded evidence that the accepted task change is represented in the certified final result and no unique unintegrated work remains.

## Remove resources

Remove the worktree first when one exists:

```text
git worktree remove <worktree-path>
```

Then delete the local branch.

Prefer:

```text
git branch -d <temporary-branch>
```

Controlled `-D` is allowed only for a proven workflow-owned branch after non-ancestry integration is explicitly verified safe.

Verify cleanup with `git worktree list` and `git branch --list`.

Every workflow-owned resource ends as **Deleted** or **Retained with reason**.