# Task Contract

Every Developer receives a bounded task contract from the controller. Keep it concrete enough to prevent scope drift while leaving implementation details to the Developer when multiple valid approaches exist.

## Required fields

### Goal

State the user-visible or architectural outcome this task must achieve.

### Scope

List the modules/files/interfaces the Developer is expected to own. If exact files are not yet certain, define the subsystem boundary instead.

### Out of scope

Call out nearby refactors, unrelated cleanup, API redesigns, dependency upgrades, or other changes that must not be performed without controller approval.

### Relevant exploration context

Provide only the Explorer findings needed for this task: call paths, data ownership, hot-file warnings, dependencies, and known risks.

### Dependencies

State whether this task:

- can start immediately;
- depends on another task/commit;
- provides an API or artifact consumed by another task;
- must not edit a shared hot file until another task completes.

### Acceptance criteria

Use observable conditions. Include behavior, compatibility, failure handling, and performance constraints when material.

### Validation

Specify the minimum relevant build target, tests, static checks, or manual verification required before handoff.

### Change permissions

Explicitly state whether the Developer may:

- add new files;
- change public APIs;
- add or upgrade dependencies;
- alter build configuration;
- modify database/schema/protocol formats;
- make cross-module refactors.

If not stated, the Developer should prefer the narrowest change and escalate material scope expansion to the controller.

## Developer handoff format

Require the Developer to return:

1. implementation summary;
2. changed and added files;
3. important design decisions;
4. build/test commands and results;
5. known limitations or residual risks;
6. deviations from the task contract, if any;
7. commit/branch/worktree state when applicable.
