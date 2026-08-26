# Task Contract

Every Developer receives a bounded task contract from the Controller. Keep it concrete enough to prevent scope drift while leaving implementation details to the Developer when multiple valid approaches exist.

Read `code-state.md` before assigning implementation work.

## Required fields

### Goal

State the user-visible or architectural outcome this task must achieve.

### Request type

Identify the task as **Feature**, **Bugfix**, **Refactor**, **Performance**, or **Maintenance**.

For Bugfix tasks, also include the current evidence summary and root-cause status. Read `bugfix.md` before assigning implementation.

### Scope

List the modules/files/interfaces the Developer is expected to own. If exact files are not yet certain, define the subsystem boundary instead.

### Out of scope

Call out nearby refactors, unrelated cleanup, API redesigns, dependency upgrades, or other changes that must not be performed without Controller approval.

### Relevant exploration context

Provide only the Explorer or Bug Investigator findings needed for this task: call paths, data ownership, hot-file warnings, dependencies, known risks, reproduction evidence, and established root cause when applicable.

### Git base and integration fields

For every implementation task record:

- `BASE_REF` — branch/ref used to create the task branch/worktree;
- `TASK_BASE_COMMIT` — immutable commit resolved from that base at task start;
- `PREDECESSORS` — task IDs/accepted commits that must already be present;
- `INTEGRATION_TARGET` — where the accepted task will be integrated; for Complex work this should normally be the workflow staging branch, not the user's target branch.

A dependent task must not start until its required predecessor outputs are accepted and available in its chosen base.

Do not silently change the task base after implementation begins. If the base must change materially, the Controller must record the new state and invalidate/re-run affected validation/review gates.

### Dependencies

State whether this task:

- can start immediately;
- depends on another accepted task/commit;
- provides an API or artifact consumed by another task;
- must not edit a shared hot file until another task completes;
- must be integrated before/after another task.

### Acceptance criteria

Use observable conditions. Include behavior, compatibility, failure handling, and performance constraints when material.

For Bugfix tasks, acceptance criteria should also state:

- the reported symptom that must no longer occur;
- the established root cause or explicitly declared mitigation/hypothesis status;
- how regression will be verified;
- adjacent behavior that must remain unchanged.

### Validation

Specify the minimum relevant build target, tests, static checks, regression test, stress test, or manual verification required before handoff.

Validation must be performed against the final committed `TASK_HEAD`, not an older working-tree snapshot.

For Bugfix, prefer a regression test that fails before the fix and passes after it whenever practical. If infeasible, define the substitute evidence required.

### Change permissions

Explicitly state whether the Developer may:

- add new files;
- change public APIs;
- add or upgrade dependencies;
- alter build configuration;
- modify database/schema/protocol formats;
- make cross-module refactors;
- add diagnostic logging or instrumentation.

If not stated, the Developer should prefer the narrowest change and escalate material scope expansion to the Controller.

## Developer handoff gate

The handoff is not valid until:

1. all intended task changes are committed;
2. the task working tree is clean;
3. `TASK_HEAD = HEAD` is recorded;
4. required scoped validation passed on that exact commit;
5. `VALIDATED_HEAD = TASK_HEAD` is recorded.

Require every Developer to return:

1. implementation summary;
2. changed and added files;
3. important design decisions;
4. `BASE_REF` and `TASK_BASE_COMMIT`;
5. `TASK_HEAD` commit SHA;
6. confirmation that the task working tree is clean;
7. build/test/check commands and results;
8. `VALIDATED_HEAD` SHA;
9. known limitations or residual risks;
10. deviations from the task contract, if any;
11. branch/worktree state when applicable.

For Bugfix tasks, additionally require:

12. reported symptom;
13. root cause;
14. evidence supporting the root cause;
15. how the fix breaks the causal chain;
16. regression verification and result;
17. precise completion state: **Confirmed fix**, **Mitigation**, **Diagnostic change**, or **Hypothesis-driven fix**.

The Controller must reject a handoff whose intended code is uncommitted, whose worktree is dirty, or whose validation does not certify the reported final `TASK_HEAD`.