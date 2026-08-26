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
- when an independent Reviewer is justified, give the Reviewer a stable final diff/state; a commit boundary is useful but not mandatory unless needed for safety/auditability.

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

## Per-task validation gate

A task is ready for authoritative review only when:

1. all intended task changes are committed;
2. the task worktree is clean;
3. `TASK_HEAD` is recorded;
4. relevant build/tests/checks pass on that exact HEAD;
5. `VALIDATED_HEAD = TASK_HEAD` is recorded.

A later delivered-content change makes stale validation invalid.

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

- acceptance criteria met;
- `TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD`;
- worktree clean;
- BLOCKER/HIGH findings resolved;
- material disputes/scope drift dispositioned.

Merge only `ACCEPTED_HEAD`.

## Integration strategy

Prefer normal merge-based integration for workflow task branches when repository policy allows because provenance/cleanup are easier to prove.

If squash/rebase/cherry-pick is required, record the integration method and evidence mapping the accepted change into staging.

## Final ORCHESTRATED gate

After accepted tasks are in staging:

1. record `STAGING_HEAD`;
2. perform clean/fresh build as appropriate;
3. run project-appropriate full tests;
4. run relevant integration/regression/stress checks;
5. set `STAGING_VALIDATED_HEAD = STAGING_HEAD` when validation passes;
6. run Integration Review over `STAGING_BASE_COMMIT..STAGING_HEAD`;
7. set `STAGING_REVIEWED_HEAD = STAGING_HEAD` when review passes;
8. if staging changes, repeat stale gates;
9. reconcile user-target drift in staging if needed and re-run gates;
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