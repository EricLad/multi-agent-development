# Task Contract

Every Developer receives a bounded task contract from the Controller. Keep it concrete enough to prevent scope drift while leaving implementation details to the Developer when multiple valid approaches exist.

Read `code-state.md` and `model-routing.md` before assigning implementation work.

## Required fields

### Goal

State the user-visible or architectural outcome this task must achieve.

### Request type

Identify the task as **Feature**, **Bugfix**, **Refactor**, **Performance**, or **Maintenance**.

For Bugfix tasks, also include the current evidence summary and root-cause status. Read `bugfix.md` before assigning implementation.

### Role, risk, and model assignment

Record the execution tier separately from Simple/Complex classification:

- `ROLE` — Developer, Bug Investigator, Explorer, Reviewer, or another explicit delegated role;
- `RISK_LEVEL` — **Low**, **Medium**, **High**, or **Critical**;
- `ASSIGNED_MODEL` — selected model tier according to `model-routing.md`;
- `REASONING_EFFORT` — intended reasoning level when configurable;
- `MODEL_RATIONALE` — required when deviating from the normal role default;
- `ESCALATION_REASON` — populate if new evidence causes an upgrade during execution.

Risk classification must consider blast radius and semantic risk, not line count. Security, concurrency/lifetime, persistent data, schema/protocol, compatibility, or broad public-API changes should not be labeled Low merely because the diff is small.

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
4. `RISK_LEVEL`, `ASSIGNED_MODEL`, and any `ESCALATION_REASON`;
5. `BASE_REF` and `TASK_BASE_COMMIT`;
6. `TASK_HEAD` commit SHA;
7. confirmation that the task working tree is clean;
8. build/test/check commands and results;
9. `VALIDATED_HEAD` SHA;
10. known limitations or residual risks;
11. deviations from the task contract, if any;
12. branch/worktree state when applicable.

For Bugfix tasks, additionally require:

13. reported symptom;
14. root cause;
15. evidence supporting the root cause;
16. how the fix breaks the causal chain;
17. regression verification and result;
18. precise completion state: **Confirmed fix**, **Mitigation**, **Diagnostic change**, or **Hypothesis-driven fix**.

The Controller must reject a handoff whose intended code is uncommitted, whose worktree is dirty, or whose validation does not certify the reported final `TASK_HEAD`.
