# Quality Gates and Git / Worktree Safety

## Validation gates

### Per-task gate

A Developer must complete the task's required scoped validation before independent review:

1. build the relevant target(s);
2. run the relevant tests;
3. run required static/lint/format checks if the project uses them;
4. report exact failures or skipped checks.

A Reviewer may request additional targeted validation when a finding depends on runtime behavior or an edge case.

### Final complex-task gate

After integration:

1. perform a clean or appropriately fresh build when project cost permits;
2. run the project-appropriate full test suite;
3. run integration/regression tests relevant to changed boundaries;
4. run Integration Review over `BASE_COMMIT..FINAL_HEAD`;
5. repeat affected gates after fixes.

Do not fabricate commands. Derive them from repository documentation, existing CI, build files, or project conventions.

## Baseline

For complex work, record `BASE_COMMIT` before implementation starts. Keep the final review range stable even if multiple worker branches are merged.

## Worktree and branch ownership ledger

Whenever this workflow creates an isolated worktree, the Controller must record enough information to clean up both Git resources later. At minimum track:

- task ID;
- worktree path;
- temporary branch name;
- integration target branch;
- whether the branch/worktree was created by this workflow;
- current lifecycle state such as active, review, accepted, merged, or cleanup-pending.

Prefer creating a worktree and its task branch as one explicit operation so ownership is unambiguous, for example conceptually:

```text
git worktree add -b <temporary-task-branch> <worktree-path> <base-ref>
```

Do not infer ownership later from branch names alone. A branch is eligible for automatic cleanup only when the workflow recorded that it created that branch for the current task.

## Worktree rules

- Use one isolated worktree/branch per independently parallelized implementation task.
- Name branches/worktrees clearly enough to map them back to task IDs.
- Do not create worktrees for tasks that cannot safely run in parallel merely for uniformity.
- Do not let multiple Developers unknowingly own the same hot file or shared API.
- Keep each worktree focused on its task contract.
- Do not delete a worktree containing uncommitted or unmerged work.
- Never delete or reset a branch/worktree that predates this workflow unless the user explicitly requests it.

## Merge rules

Before merging a worker branch, the controller confirms:

- task acceptance criteria are met;
- scoped build/tests passed;
- BLOCKER/HIGH findings are resolved;
- material disputes are arbitrated;
- scope drift is understood and accepted.

When resolving merge conflicts, preserve task semantics rather than choosing a side mechanically. If a conflict changes behavior or shared interfaces, revalidate affected tasks.

After a task branch is integrated, mark its ledger entry as `merged` or otherwise record the integration method. Do not delete its branch/worktree until the final acceptance gate permits cleanup.

## Mandatory cleanup gate

Cleanup is part of task completion, not an optional cosmetic step.

After final acceptance, process every workflow-owned temporary worktree/branch pair individually.

### 1. Confirm ownership

Delete automatically only if the ledger says the worktree and branch were created by this workflow for the current task.

Never automatically delete:

- the user's pre-existing branches or worktrees;
- the integration target branch;
- branches whose ownership is uncertain;
- remote branches merely because a similarly named local temporary branch exists.

### 2. Confirm the worktree is clean

Check that the temporary worktree has no uncommitted changes before removal. A typical check is:

```text
git -C <worktree-path> status --porcelain
```

The expected result is empty output. If the worktree is dirty, stop cleanup for that entry and report it. Do not use force removal just to make cleanup succeed.

### 3. Confirm the branch is integrated

For a normal merge-based integration, verify that the temporary branch tip is an ancestor of the final integration target, for example:

```text
git merge-base --is-ancestor <temporary-branch> <integration-target>
```

Exit status 0 is strong evidence that the branch has been merged.

If the workflow intentionally used squash, rebase, cherry-pick, or another integration method where ancestry is not preserved, do not pretend the ancestry check succeeded. Delete the branch automatically only when the Controller has explicit recorded evidence that the task's intended changes were integrated and validated. If that evidence is uncertain, preserve the branch and report why cleanup was deferred.

### 4. Remove the temporary worktree

After the safety checks pass:

```text
git worktree remove <worktree-path>
```

Do not use `--force` by default.

A branch that is still checked out by a worktree cannot normally be deleted, so worktree removal comes before local branch deletion.

### 5. Delete the workflow-created temporary local branch

Once the worktree is removed, delete the corresponding local branch using Git's safe deletion mode:

```text
git branch -d <temporary-branch>
```

Prefer `-d`, not `-D`. If Git refuses deletion because it considers the branch unmerged, do not force-delete it automatically. Re-check integration evidence and either resolve the lifecycle state or preserve the branch and report the reason.

### 6. Verify cleanup

Do not assume commands succeeded. Verify both resources are gone, for example with:

```text
git worktree list
git branch --list <temporary-branch>
```

The worktree path should no longer be listed and the branch query should return no matching local branch.

`git worktree prune` may be used to remove stale administrative metadata when appropriate, but it is not a substitute for deleting the temporary task branch.

## Cleanup completion condition

At the end of a complex workflow, every workflow-owned temporary resource must be in one of these states:

- **Deleted** — temporary worktree removed and its local task branch deleted; or
- **Retained with reason** — cleanup was intentionally blocked by dirty state, uncertain integration, audit/PR requirements, or another concrete safety reason reported to the user.

Do not declare cleanup complete merely because all worktree directories were removed while workflow-created temporary local branches remain silently in the repository.
