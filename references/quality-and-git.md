# Quality Gates and Git / Worktree Safety

Read `code-state.md` before implementation starts. All validation, review, integration, and cleanup decisions are bound to exact commits.

## Git preflight gate

Before any Developer edits code, the Controller records the repository state required by `code-state.md`:

- user target branch and its starting HEAD;
- whether the current working tree is dirty;
- existing worktrees;
- existing local branches relevant to workflow naming.

Do not mix workflow edits with pre-existing uncommitted user changes. Preserve user state and isolate workflow work when necessary.

## Per-task validation gate

A Developer is not ready for authoritative review until:

1. all intended task changes are committed;
2. the task working tree is clean;
3. `TASK_HEAD` is recorded;
4. the relevant build target(s) pass on that exact `TASK_HEAD`;
5. relevant tests pass on that exact `TASK_HEAD`;
6. required static/lint/format checks pass when the project uses them;
7. `VALIDATED_HEAD = TASK_HEAD` is recorded.

If code, tests, build files, configuration, schema, protocol, or other task-relevant content changes afterward, the prior validation is stale.

Do not fabricate commands. Derive them from repository documentation, CI, build files, or established project conventions.

## Workflow baseline and staging branch

For Complex work:

- record `TARGET_BASE_COMMIT` before implementation;
- create a workflow-owned staging/integration branch from that commit;
- record the staging branch in the Git resource ledger;
- integrate only Controller-accepted task commits into staging;
- keep the user's target branch at its last known good state until the staging result has passed final gates.

The final Integration Review range is based on the staging baseline and the exact current staging HEAD.

## Git resource ledger

Track **every Git resource created by this workflow**, not only resources that have worktrees.

At minimum record:

- resource/task ID;
- local branch name;
- worktree path when one exists;
- resource role: task branch, simple-task branch, staging branch, or other workflow-owned temporary branch;
- base ref and base commit;
- predecessor/dependency information when applicable;
- integration target;
- whether the branch/worktree was created by this workflow;
- lifecycle state such as active, review, accepted, integrated, cleanup-pending, deleted, or retained-with-reason.

Do not infer ownership later from branch names alone.

## Worktree rules

- Use one isolated worktree/branch per independently parallelized implementation task.
- Name branches/worktrees clearly enough to map them to task IDs.
- Do not create worktrees for tasks that cannot safely run in parallel merely for uniformity.
- Do not let multiple Developers unknowingly own the same hot file or shared API.
- Keep each worktree focused on its task contract.
- Do not delete a worktree containing uncommitted work.
- Never delete or reset a branch/worktree that predates this workflow unless the user explicitly requests it.

When creating a task worktree, use the explicit `BASE_REF` / `TASK_BASE_COMMIT` determined by the dependency plan. Conceptually:

```text
git worktree add -b <temporary-task-branch> <worktree-path> <base-ref>
```

Record the resulting task base immediately.

## Dependency-aware branch creation

A task with predecessors does not become ready merely because an agent slot is available.

Before creating/starting it:

- all required predecessor outputs must be accepted;
- the selected `BASE_REF` must contain the predecessor commits required by the contract;
- record `TASK_BASE_COMMIT` from that exact base;
- do not silently rebase the task later onto a different base without invalidating and re-running the relevant task gates.

Integrate accepted tasks into staging in topological/dependency order.

## Per-task merge gate

Before a task is eligible for staging integration, verify:

- acceptance criteria are met;
- `TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD`;
- the task working tree is clean;
- BLOCKER/HIGH findings are resolved;
- material disputes are arbitrated;
- scope drift is understood and accepted.

Merge only `ACCEPTED_HEAD`, never an unreviewed or unvalidated later task tip.

When resolving task-to-staging merge conflicts, preserve task semantics rather than choosing a side mechanically. Any semantic conflict resolution changes `STAGING_HEAD` and must be covered by the final staging gates.

## Integration strategy

For workflow-created temporary task branches, prefer normal merge-based integration that preserves ancestry when repository policy allows it. This makes provenance and safe cleanup verifiable.

If repository policy or the user explicitly requires squash, rebase, cherry-pick, or another non-ancestry-preserving strategy:

- record that integration method in the resource ledger;
- record explicit evidence mapping the accepted task change into staging;
- do not pretend ancestry-based checks apply;
- use the special cleanup rule below for the temporary branch.

## Final Complex gate — one canonical order

After all intended accepted tasks are integrated into the workflow staging branch:

1. record `STAGING_HEAD`;
2. perform the clean/appropriately fresh build required by the project;
3. run the project-appropriate full test suite;
4. run relevant integration/regression/stress checks;
5. if all required validation passes, set `STAGING_VALIDATED_HEAD = STAGING_HEAD`;
6. run the independent Integration Reviewer over `STAGING_BASE_COMMIT..STAGING_HEAD`, with validation results available as context;
7. if review passes, set `STAGING_REVIEWED_HEAD = STAGING_HEAD`;
8. if a repair changes staging, invalidate stale validation/review and repeat until the same exact staging HEAD is both validated and reviewed;
9. check for user-target drift and incorporate the latest target into staging if necessary;
10. if staging changes due to target drift/conflict resolution, repeat final validation and Integration Review;
11. only then promote the certified staging result into the user target branch.

The canonical readiness invariant is:

`STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD`

Do not use the conflicting order `Integration Review -> tests` in one place and `tests -> Integration Review` in another. This file defines the canonical order.

## Promotion to the user target

Keep the user target at its last known good state until staging has passed the canonical final gate.

Before promotion, verify whether the user target moved since `TARGET_BASE_COMMIT`. Follow the drift procedure in `code-state.md` when it did.

Promotion should not introduce unreviewed tree changes. If the final merge into the user target requires semantic conflict resolution, perform that resolution in staging and rerun the final gates rather than resolving directly on the user target and declaring success.

## Mandatory cleanup gate

Cleanup is part of task completion, not an optional cosmetic step. Process **every workflow-owned temporary Git resource**, including:

- parallel task worktrees and branches;
- Simple-task temporary branches even when no worktree was created;
- the workflow staging branch;
- any other branch explicitly recorded as workflow-owned.

### 1. Confirm ownership

Delete automatically only when the resource ledger proves the branch/worktree was created by this workflow for the current run.

Never automatically delete:

- user pre-existing branches/worktrees;
- the user target branch;
- branches whose ownership is uncertain;
- remote branches merely because a similarly named local temporary branch exists.

### 2. Confirm worktree cleanliness when applicable

For a resource with a worktree:

```text
git -C <worktree-path> status --porcelain
```

Expected output is empty. If dirty, retain the resource and report why. Do not force-remove it merely to complete cleanup.

### 3. Confirm integration

For normal merge-based integration, verify the temporary branch tip is an ancestor of its final integration target, for example:

```text
git merge-base --is-ancestor <temporary-branch> <integration-target>
```

For squash/rebase/cherry-pick/non-ancestry integration, require recorded evidence that:

- the branch is workflow-owned;
- its `ACCEPTED_HEAD` change is present in the certified staging/final result;
- the final result passed validation and review;
- no unique unintegrated commits/worktree changes remain on the temporary resource.

If any of those facts are uncertain, retain the branch and report why.

### 4. Remove the worktree when one exists

After safety checks pass:

```text
git worktree remove <worktree-path>
```

Do not use `--force` by default.

### 5. Delete the workflow-created local branch

For merge-based integration, prefer Git safe deletion:

```text
git branch -d <temporary-branch>
```

For an explicitly recorded non-ancestry integration where `-d` rejects the branch even though all special safety proofs above are satisfied, the Controller may use controlled force deletion:

```text
git branch -D <temporary-branch>
```

`-D` is allowed only for a branch proven to be workflow-owned and fully represented in the already certified final result. Never use it merely to make the repository look clean.

### 6. Verify cleanup

Verify commands actually removed the resources:

```text
git worktree list
git branch --list <temporary-branch>
```

A worktree path should no longer be listed and a deleted local branch query should return no match.

`git worktree prune` may clean stale metadata, but it is not a substitute for branch deletion.

## Cleanup completion condition

Every workflow-owned temporary resource must end in exactly one of these states:

- **Deleted** — the temporary worktree, if any, and the local branch are removed and verified absent; or
- **Retained with reason** — cleanup is intentionally blocked by dirty state, uncertain integration, PR/audit requirements, or another concrete safety reason reported to the user.

Do not declare cleanup complete while workflow-created local branches remain silently in the repository.