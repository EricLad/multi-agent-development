# Agent Pool and Concurrency Scheduling

Use the Agent Pool only for **ORCHESTRATED work where more than one subagent may be active**.

Do not use Agent Pool bookkeeping for FAST. Do not use it for a normal one-Developer STANDARD task.

## Principle

The goal is reliable parallel throughput, not maximum fan-out.

**Schedule to shorten the critical path, not to maximize active-agent count.**

Do not rely on the current model to volunteer parallelism. When two or more independent Ready tasks lie on, unblock, or materially shorten the critical path, the Controller should explicitly consider parallel delegation and use it when expected wall-clock savings exceed coordination, review, build, and merge costs.

Do not hardcode a universal Codex concurrency limit. If the runtime exposes a limit, respect it. Otherwise start conservatively and adapt from observed capacity.

## What consumes capacity

Count active subagents such as:

- Explorer;
- Bug Investigator;
- Developer;
- Reviewer;
- repair Developer;
- Integration Reviewer.

The main Controller is not counted unless the runtime says otherwise.

## Default when capacity is unknown

- start with at most 3 concurrent implementation Developers;
- keep practical capacity for review/investigation/repair;
- increase only when runtime capacity and task independence are both demonstrated.

These are scheduling defaults, not product-limit claims.

## Queue state

Track only what is needed:

- **Active** — currently consuming slots;
- **Ready** — dependency-satisfied work that may start;
- **Blocked** — waiting on predecessor, hot-file ownership, review, or decision.

Do not spawn all decomposed tasks merely because they exist.

## Critical-path scheduling

When Explorer/Controller identifies a critical path:

1. keep critical-path tasks unblocked whenever possible;
2. explicitly parallelize independent Ready work when it will shorten the critical path and coordination cost is lower than the expected saving;
3. start off-path parallel work only when it will not delay critical-path review, repair, build, or integration;
4. avoid speculative fan-out whose output cannot be integrated until a predecessor stabilizes;
5. prefer one clear owner over parallel work that creates hot-file or shared-interface churn.

Agent availability alone is not sufficient reason to start a task.

## Scheduling priority

Prefer:

1. unblocking investigation/exploration on the critical path;
2. Reviewer for completed critical-path implementation;
3. repair work for blocking findings;
4. critical-path Developer task;
5. other independent Developer work;
6. optional analysis.

This prevents review starvation and reduces wall-clock delay on the path that determines completion.

## Pipeline review

When a Developer completes a committed/validated ORCHESTRATED handoff, release its active execution slot when possible while retaining its branch/worktree and role-owned context if compatible repair work may return to it.

Start the Reviewer when capacity allows while other independent Developers continue.

Reviewer findings should normally arrive as one batch. Prefer one grouped repair pass for compatible findings rather than repeatedly occupying slots for tiny serial repair cycles. When practical, return that repair to the same Developer context because it already owns the implementation evidence; create a fresh repair agent only when independence, availability, isolation, or lost context justifies it.

Agent execution lifecycle, retained context, and worktree lifecycle are related but not identical. Do not delete useful task state merely to free an active slot when the runtime allows safe retention.

## Convergence handling

Track the number of **material review/repair cycles** per task.

Expected shape:

`implementation -> complete review -> batch repair -> re-review -> converge`

If substantial new BLOCKER/HIGH/MEDIUM findings continue after a second material repair/re-review cycle:

- stop automatically scheduling another repair loop;
- mark the task `Blocked: Convergence Gate`;
- return it to the Controller for plan/invariant/task-boundary/model-capability review;
- resume only after the Controller explicitly replans, splits, defers scope, escalates reasoning when justified, or authorizes another cycle.

This prevents review backlog and agent capacity from being consumed by unbounded hardening loops.

## Thread-limit handling

If spawning fails because the runtime limit is reached:

1. do not duplicate the task;
2. release completed active agents when safe;
3. return the activity to Ready;
4. lower observed concurrency if necessary;
5. retry only after capacity becomes available.

A pool-limit error is a scheduler event, not task failure.

## Dynamic reduction

Reduce concurrency when:

- hot-file overlap appears;
- dependency assumptions fail;
- merge conflicts increase;
- review backlog grows;
- repair churn grows;
- build/test resource contention harms reliability;
- runtime repeatedly hits the limit.

If builds/tests are the bottleneck, more Developers can make total wall-clock time worse. Reduce fan-out accordingly.

## Integration phase

Before final Integration Review:

- stop unnecessary new implementation work;
- release completed idle agents;
- ensure capacity for Integration Reviewer and possible batch repair;
- avoid starting off-path optional work that can delay final staging validation.

## Completion invariant

Parallelism must not weaken ORCHESTRATED quality gates. Throughput is secondary to correctness, but unnecessary fan-out is not a quality feature.
