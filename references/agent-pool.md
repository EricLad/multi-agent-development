# Agent Pool and Concurrency Scheduling

Use an **Agent Pool / Concurrency Gate** whenever the workflow may run more than one subagent. The goal is reliable throughput, not maximum instantaneous fan-out.

Read `code-state.md` when scheduling implementation/review transitions so agent lifecycle decisions do not weaken commit-bound quality gates.

## Do not hardcode a global limit

Codex concurrency can vary by client, version, account, runtime, or future product changes. Do not assume that a fixed number such as 3, 4, or 6 is universally available.

If the environment exposes a concrete concurrent-agent limit, use that value. If it does not, use conservative scheduling and adapt only from observed successful capacity.

A failure to spawn because the agent/thread limit is reached is a **scheduler event**, not an implementation failure.

## What consumes pool capacity

Count every active subagent, not only Developers. Depending on the current phase this can include:

- Explorer;
- Bug Investigator;
- Developer;
- Reviewer;
- repair/rework Developer;
- Integration Reviewer;
- other explicitly delegated diagnostic agents.

The Controller/main session is not counted as a subagent slot unless the runtime explicitly says otherwise.

## Default policy when capacity is unknown

When the runtime does not expose its limit:

1. Start conservatively with at most **3 concurrent implementation Developers**.
2. Keep capacity available for Reviewers, investigation, or repair work instead of filling every observed slot with Developers.
3. If the environment clearly demonstrates that more concurrent subagents are supported and additional parallelism is safe, the Controller may cautiously increase the implementation wave size, normally to **4** before considering anything higher.
4. Never increase concurrency merely because many tasks exist. Parallelism must still satisfy dependency and hot-file rules.

These values are scheduling defaults, not claims about Codex's product limit.

## Reserve capacity

When a concrete pool capacity is known, avoid consuming the entire pool with implementation workers.

Use this heuristic unless the environment or task requires otherwise:

- capacity `<= 3`: reserve no fixed slot, but serialize aggressively and immediately recycle completed agents;
- capacity `4-5`: normally reserve **1** slot for review/investigation/repair;
- capacity `>= 6`: normally reserve **2** slots for review/investigation/repair.

The Controller may temporarily use a reserved slot for a Developer only when no review/investigation work is pending and doing so will not prevent near-term quality gates from running.

## Queue and waves

Maintain three pieces of state for complex work:

- **Active agents** — currently consuming subagent capacity;
- **Ready queue** — dependency-satisfied tasks that may start when a slot is available;
- **Blocked queue** — tasks waiting on another accepted task/commit, shared API, hot-file owner, review result, or Controller/user decision.

Do not spawn all decomposed tasks at once. Launch them in **waves** according to available capacity and dependency safety.

A task is not ready merely because a slot exists. Its predecessor and `BASE_REF` requirements from `task-contract.md` must also be satisfied.

Example:

```text
Ready: A B C D E F G
Known capacity: 6
Reserved quality slots: 2

Wave 1 implementation:
Developer A
Developer B
Developer C
Developer D

Reserved / reusable:
2 slots for Reviewer, Investigator, repair, or later work
```

When `Developer A` finishes a valid handoff:

```text
all intended changes committed
working tree clean
TASK_HEAD recorded
VALIDATED_HEAD == TASK_HEAD
release Developer A agent slot
keep Worktree A
start Reviewer A if review is ready
```

A worktree is repository state; an agent slot is runtime capacity. Treat them independently.

## Scheduling priority

When multiple ready activities compete for a slot, use this default priority:

1. **Unblocking investigation/exploration** needed to make other tasks safe;
2. **Reviewer / re-Reviewer** for a committed validated implementation;
3. **Repair Developer** for BLOCKER/HIGH findings or Controller-required fixes;
4. **New Developer** task from the ready queue;
5. Optional additional analysis or non-blocking checks.

This prevents the pool from becoming saturated with new code while completed code waits indefinitely for review.

## Pipeline review instead of batch review

Do not wait for every Developer to finish before starting all reviews.

Prefer pipelining:

```text
Developer A completes commit-bound handoff
  -> release A's agent slot
  -> Reviewer A reviews TASK_BASE_COMMIT..TASK_HEAD

Developer B/C/D may continue in parallel
```

If Reviewer A finds a blocking defect, route the result back to the original Developer identity/task when practical. Reopen or reassign an implementation agent only when a slot is available.

After any repair changes the task HEAD, the old review approval is stale. Schedule re-review of the new validated HEAD before task acceptance.

## Agent lifecycle

After an implementation agent completes its current responsibility:

- collect the handoff and exact commit/validation evidence;
- require all intended changes to be committed;
- require the worktree to be clean;
- verify `VALIDATED_HEAD == TASK_HEAD` before considering the implementation ready for review;
- release/close the agent thread when the runtime supports explicit lifecycle control;
- retain its branch/worktree until review, repair, Controller acceptance, staging integration, and cleanup are complete.

Do not accept "otherwise safely preserved" uncommitted implementation state as equivalent to a commit. Branch-based integration and review require a committed snapshot.

Do not keep completed agents alive merely to preserve their worktree.

Do not delete a worktree merely to free an agent slot.

## Thread-limit handling

If spawning a subagent fails because a concurrent-agent/thread limit was reached:

1. do not duplicate the task in another agent;
2. inspect which active agents have completed and have valid committed handoffs;
3. place the requested activity back in the appropriate ready queue;
4. release completed agents where possible;
5. retry only after capacity becomes available;
6. if observed capacity is lower than expected, reduce the concurrency target for the remainder of the workflow.

Do not interpret a pool-limit error as evidence that the task itself failed.

## Dynamic adjustment

The Controller should reduce concurrency when:

- hot-file overlap appears;
- dependency assumptions prove wrong;
- merge conflicts increase;
- review/re-review backlog grows;
- the runtime repeatedly hits its agent limit;
- build/test resource contention makes validation unreliable;
- multiple workers need the same external resource or mutable environment.

The Controller may increase concurrency when:

- capacity is confirmed;
- tasks have independent ownership boundaries;
- dependency prerequisites are already satisfied;
- the ready queue is large enough to benefit;
- review and repair still have available capacity;
- build/test infrastructure can sustain the parallel load.

## Integration phase

Before final staging validation and Integration Review:

- stop launching new implementation work unless required to fix an accepted finding;
- release completed per-task agents that no longer need to remain active;
- integrate only task `ACCEPTED_HEAD` commits into the workflow staging branch in dependency order;
- preserve enough capacity for the Integration Reviewer and any repair/revalidation/re-review cycle.

The Integration Reviewer should not compete with unnecessary idle Developer agents for the last available slot.

## Controller status tracking

For complex work, keep a lightweight internal table equivalent to:

| Task | State | Agent role | Base/Head | Worktree | Dependency | Review |
| --- | --- | --- | --- | --- | --- | --- |
| A | implementing | Developer | baseA / headA | wt-a | none | pending |
| B | review | Reviewer | baseB / headB | wt-b | none | active |
| C | blocked | — | pending | not created | A accepted API | pending |
| D | ready | — | baseD / — | not created | none | pending |

The exact representation is flexible. The invariant is that the Controller knows what consumes capacity, what exact code snapshot is being certified, what can start next, and what is blocked.

## Completion invariant

Concurrency optimization must never weaken the existing quality gates:

- Developer != Reviewer;
- implementation state is committed before review;
- validation and review certify exact commits;
- code changes invalidate stale validation/review approvals;
- blocking findings require repair or Controller arbitration;
- complex work is certified on staging before promotion to the user target;
- final staging validation precedes Integration Review and both must certify the same HEAD.

Throughput is secondary to correctness and review independence.