# Role + Risk Adaptive Model Routing

Use adaptive model routing instead of assigning one fixed model to every subagent. The Controller chooses the least expensive model that is appropriate for the role and risk, then escalates when evidence shows that the current tier is insufficient.

Model choice is a quality-control decision. Do not downgrade a high-risk task merely to increase parallel throughput or reduce cost.

## Model families

When these configurations are available, use the following capability tiers:

- **Luna max** — cost-efficient, high-volume work with strong reasoning effort; preferred for exploration, routine review, and bounded low-risk implementation.
- **Terra xhigh** — default production implementation tier; balances capability and cost for normal software engineering.
- **Terra max** — high-risk implementation, difficult root-cause analysis, concurrency/lifetime/state-machine work, or other tasks where additional reasoning is justified.
- **Sol xhigh / max** — escalation tier for critical, unusually difficult, or repeatedly unresolved work where frontier capability is materially valuable.

If an exact model or reasoning effort is unavailable, use the closest available capable configuration while preserving the intended capability ordering. State a material downgrade once; do not silently weaken a quality-critical role.

## Risk is separate from Simple / Complex

Execution complexity and model risk are related but not identical.

A task may be structurally Simple but still high risk, for example:

- a one-line authentication or authorization change;
- a small memory-lifetime fix in a critical callback;
- a schema/protocol compatibility change;
- a tiny change that can corrupt persistent data.

Conversely, a Complex task may contain many low-risk exploration or mechanical implementation slices.

For every implementation/investigation task, classify `RISK_LEVEL` independently as **Low**, **Medium**, **High**, or **Critical**.

## Risk signals

### Low

Typical characteristics:

- localized and reversible;
- no public API/schema/protocol/security/lifetime/concurrency impact;
- deterministic validation is straightforward;
- failure blast radius is small;
- implementation is mechanical or strongly patterned by nearby code.

### Medium

Typical characteristics:

- normal production feature/refactor work;
- some cross-file/module reasoning;
- moderate regression surface;
- existing architecture must be understood and preserved;
- mistakes are recoverable and well-covered by tests/review.

### High

Any of the following is a strong signal:

- concurrency, async ordering, ownership, object lifetime, memory/resource safety;
- authentication, authorization, secrets, trust boundaries, or other security-sensitive behavior;
- database/schema/migration or persistent-data correctness;
- public API, protocol, serialization, compatibility, or build-system changes with broad consumers;
- cross-module architecture or state-machine behavior;
- difficult intermittent bug diagnosis;
- large blast radius or expensive rollback;
- multiple plausible root causes with weak discriminating evidence.

### Critical

Use sparingly when failure could cause severe data loss/security impact, the architecture is unusually difficult, or lower tiers have failed to converge on a defensible solution.

## Role routing

### Explorer

Default:

`Explorer -> Luna max`

Exploration is normally read-only and high-volume: repository discovery, call chains, dependency mapping, hot-file analysis, build/test discovery, and task decomposition support.

Escalate to **Terra xhigh** when exploration itself requires difficult architectural inference, very long-context synthesis, conflicting subsystem behavior, or ambiguous ownership/lifecycle reasoning.

### Bug Investigator

Start with:

`Low/Medium diagnostic work -> Luna max`

Use:

`High-risk or non-trivial root-cause work -> Terra xhigh`

Escalate to:

`Concurrency/lifetime/state corruption or stubborn multi-hypothesis diagnosis -> Terra max`

Use **Sol xhigh/max** only when the investigation remains materially unresolved after strong evidence gathering, or the bug is Critical.

Do not keep a weak diagnosis merely to avoid escalation.

### Developer

Default routing:

- **Low-risk Simple implementation -> Luna max**
- **Medium / normal production implementation -> Terra xhigh**
- **High-risk implementation -> Terra max**
- **Critical implementation -> Sol xhigh/max** when available and justified

Luna max is not the default for ordinary production coding. Use it when the task contract is narrow, the implementation is strongly patterned, and validation/review can reliably detect mistakes.

A small diff does not automatically qualify for Luna. Security, lifetime, concurrency, persistence, protocol, or compatibility risk overrides diff size.

### Reviewer

Default:

`Per-task Reviewer -> Luna max`

The Reviewer remains independent from the Developer and certifies an exact commit as defined by `code-state.md`.

Escalate the review to **Terra xhigh** when:

- the task is High/Critical risk;
- the Developer used Terra max/Sol for difficult reasoning;
- a material Developer/Reviewer dispute cannot be resolved confidently;
- the patch changes security, concurrency/lifetime, schema/protocol, or broad public behavior;
- the first review reports uncertainty about correctness rather than a clear pass/finding set.

For Critical changes, the Controller may require a second independent high-tier review instead of merely replacing the first review.

### Integration Reviewer

Default:

`Integration Reviewer -> Terra xhigh`

Integration Review has a larger cognitive surface than ordinary per-task review because it must reason across multiple accepted tasks, merge interactions, lifecycle/state assumptions, and final staging behavior.

Escalate to **Terra max** when the integrated change is High risk, spans many coupled subsystems, contains meaningful merge/conflict resolution, or includes concurrency/security/data-migration concerns.

Escalate to **Sol xhigh/max** for Critical integration or when Terra-level integration review remains materially uncertain.

## Escalation triggers

Escalate one capability tier when any of these occur and the stronger tier is likely to improve the result:

- the agent cannot establish a defensible answer after reasonable repository investigation;
- root-cause confidence remains low with multiple plausible candidates;
- repeated implementation/test cycles fail without convergence;
- a Reviewer identifies architecture-level uncertainty rather than a localized defect;
- Developer and Reviewer produce a material evidence-based dispute the Controller cannot resolve confidently;
- unexpected hot-file/dependency/cross-module coupling appears;
- validation exposes intermittent or non-local failures;
- target-drift or merge conflict resolution creates a materially more complex integrated state;
- the task's actual risk is higher than initially classified.

Do not escalate merely because a task is long. Escalate for reasoning difficulty, risk, uncertainty, or failure to converge.

## De-escalation and cost control

Use stronger models only for the phase that needs them.

Examples:

- a Terra max Bug Investigator can establish the root cause, while a narrow Low-risk follow-up implementation may still use Terra xhigh or Luna max if the task contract makes the implementation mechanical;
- a Terra max Developer does not imply every documentation/test bookkeeping subtask also needs Terra max;
- after a high-tier escalation resolves an uncertainty, return unrelated ready-queue work to its normal role/risk routing.

Do not sacrifice Developer/Reviewer independence when reusing model tiers. Two agents using the same model are still separate roles/contexts.

## Task-contract fields

For every delegated implementation or investigation activity, record when applicable:

- `ROLE`;
- `RISK_LEVEL` — Low / Medium / High / Critical;
- `ASSIGNED_MODEL`;
- `REASONING_EFFORT`;
- `MODEL_RATIONALE` — short explanation when not using the role default;
- `ESCALATION_REASON` — populated if the task was upgraded during execution.

The Controller may change the assigned tier when new evidence changes risk or uncertainty, but should record why.

## Agent-pool interaction

Model routing does not change the Agent Pool correctness rules.

- every active subagent still consumes runtime capacity according to the environment;
- do not saturate the pool with cheap Luna workers and starve review/repair slots;
- a model escalation is a scheduler transition, not permission to duplicate the same task concurrently;
- preserve the same task contract, Git snapshot state, and evidence when handing an unresolved task to a stronger model.

## Completion invariant

Adaptive routing must never weaken:

- commit-bound validation;
- Developer != Reviewer independence;
- re-review after delivered-code changes;
- Bugfix root-cause evidence;
- staging Integration Review;
- final promotion and Git cleanup gates.

Cost and throughput are optimization goals. Correctness and evidence remain the acceptance criteria.
