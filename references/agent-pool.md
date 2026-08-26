# Agent Pool and Concurrency Scheduling

Use the Agent Pool only for **ORCHESTRATED work where more than one subagent may be active**.

Do not use Agent Pool bookkeeping for FAST. Do not use it for a normal one-Developer STANDARD task.

## Principle

The goal is reliable parallel throughput, not maximum fan-out.

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

## Scheduling priority

Prefer:

1. unblocking investigation/exploration;
2. Reviewer for completed implementation;
3. repair work for blocking findings;
4. new Developer task;
5. optional analysis.

This prevents review starvation.

## Pipeline review

When a Developer completes a committed/validated ORCHESTRATED handoff, release its agent slot when possible while retaining its branch/worktree. Start the Reviewer when capacity allows while other independent Developers continue.

Agent lifecycle and worktree lifecycle are independent.

## Thread-limit handling

If spawning fails because the runtime limit is reached:

1. do not duplicate the task;
2. release completed agents when safe;
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
- build/test resource contention harms reliability;
- runtime repeatedly hits the limit.

## Integration phase

Before final Integration Review:

- stop unnecessary new implementation work;
- release completed idle agents;
- ensure capacity for Integration Reviewer and possible repair.

## Completion invariant

Parallelism must not weaken ORCHESTRATED quality gates. Throughput is secondary to correctness and review independence.