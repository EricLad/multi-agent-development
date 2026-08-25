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

## Cleanup

Only after final acceptance:

- remove temporary worktrees created by this workflow;
- remove temporary task branches when safe and desired;
- preserve branches needed for PRs, auditability, or user workflows;
- report any cleanup that could not be completed safely.
