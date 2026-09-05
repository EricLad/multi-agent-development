# Capability + Cost Model Routing

Apply model routing only when subagents are actually useful. Do not create subagents or bookkeeping solely to exercise this policy.

The central rule is:

> **Risk controls governance; execution ambiguity, task horizon, integration breadth, expected iteration cost, and cost sensitivity control model choice.**

A high-risk task does not automatically need the most capable Developer when the implementation boundary is explicit. Conversely, a low-risk task may still justify a stronger model when the work is long-horizon, cross-system, or repeatedly fails to converge.

# Routing dimensions

Use only the fields that materially affect the decision:

- `RISK_LEVEL` — Low / Medium / High / Critical; drives review, validation, isolation, and approval strength.
- `EXECUTION_AMBIGUITY` — Low / Medium / High; how much consequential implementation reasoning remains.
- `TASK_HORIZON` — Short / Medium / Long; how much state and goal continuity the task requires.
- `INTEGRATION_BREADTH` — Local / Subsystem / Cross-system; how widely correctness depends on interactions.
- `COST_SENSITIVITY` — Low / Medium / High when model cost materially matters.
- `ASSIGNED_MODEL` and, when relevant, `REASONING_EFFORT`.
- `MODEL_RATIONALE` when the selected path is not obvious.

# Model families

Use current runtime availability rather than assuming every installation exposes every model.

## GPT-5.6 Terra

Use Terra as the cost/performance execution path for clear, bounded coding work.

Good fit:

- plan-ready routine features;
- bounded multi-file implementation;
- clear ORCHESTRATED subtasks;
- straightforward review repairs;
- high-risk tasks whose consequential architecture is already decided and whose implementation itself is not unusually difficult.

Default tendency: **Terra medium**.

Escalate within the GPT-5.6 family only when Astra is unavailable or a project/runtime policy specifically favors the older family.

## GPT-6 Astra

Use Astra when the dominant cost is cognition, continuity, or repeated reasoning failure rather than raw implementation volume.

Strong signals:

- long-horizon end-to-end work;
- cross-system implementation where many interacting constraints must remain active;
- materially ambiguous implementation after reasonable Controller planning;
- difficult architecture-to-code reconciliation;
- stubborn multi-hypothesis debugging;
- concurrency/lifetime/state-machine/data-integrity semantics;
- broad integration review;
- repeated non-convergence where restarting weaker agents would discard useful context.

Start with the lowest reasoning effort that is likely to solve the task. Typical progression when supported:

`Astra medium -> high -> xhigh -> max`

Do not jump directly to max merely because the task is important.

# Workflow-tier interaction

## FAST

No delegated-role routing by default. Use the current main-session model and keep the task in one coding context.

Do not create a stronger subagent merely because one exists.

## STANDARD

Use one Developer when delegation is useful.

Typical routing:

- bounded + Low execution ambiguity + Short/Medium horizon -> **Terra medium**;
- materially ambiguous or Long-horizon Subsystem work -> **Astra medium/high** when available;
- difficult unresolved semantics -> increase Astra reasoning before replacing the context when practical.

Independent review remains conditional on risk and uncertainty.

## ORCHESTRATED

Route each task independently after its boundary is ready.

Decomposition should make most implementation tasks simpler. That often keeps routine Developers on Terra even when the overall program is complex.

Use Astra selectively for tasks whose own local reasoning remains hard, for long-lived Controller/integration contexts, or for difficult diagnosis/review.

# Plan Readiness before Developer assignment

Do not compensate for an unclear consequential boundary by immediately selecting a stronger Developer.

Before starting a STANDARD or ORCHESTRATED Developer, the Controller should ensure:

1. **Goal** — required outcome is clear;
2. **Scope** — ownership/files/subsystem boundary is clear enough;
3. **Architectural decisions / hard constraints** — consequential choices the Developer must not silently change are explicit;
4. **Non-goals** — meaningful nearby work to avoid is clear;
5. **Validation** — success can be checked concretely.

Routine local tactics do not need to be pre-specified when repository evidence can resolve them safely. The Developer may choose helpers, local structure, exact edit sequence, and similar reversible details inside the accepted boundary.

# Developer routing

## Route A: bounded execution

Prefer **GPT-5.6 Terra medium** when:

- `EXECUTION_AMBIGUITY = Low`;
- horizon is Short or Medium;
- integration breadth is Local or Subsystem;
- architecture/public behavior is already constrained where necessary;
- expected rework is low.

This remains the default cost-efficient production path.

## Route B: cognition-dominant execution

Prefer **GPT-6 Astra medium/high** when one or more of these materially dominate the task:

- Long task horizon;
- Cross-system breadth;
- consequential implementation ambiguity remains after planning;
- many interacting invariants must be preserved simultaneously;
- the task requires sustained reasoning across code search, implementation, validation, and repair;
- prior lower-cost attempts failed to converge.

Choose `medium` when the task is long but structurally clear; choose `high` when consequential reasoning remains substantial.

## Route C: difficult unresolved semantics

Escalate Astra to `xhigh` or `max` only for genuinely difficult reasoning such as:

- unresolved concurrency/lifetime/resource-safety interactions;
- subtle state-machine or persistence consistency problems;
- repeated non-convergence after a sound plan and meaningful evidence;
- difficult integration conflicts where local fixes interact in non-obvious ways.

Before escalating, check whether the real problem is a bad task boundary, missing invariant, or incorrect architecture assumption.

# Reasoning-effort escalation

When the runtime can adjust reasoning effort while preserving the same useful context, prefer:

`same Astra context + higher reasoning effort`

over
`discard context + spawn a replacement agent`

when all of the following hold:

- the same role still owns the work;
- the current context contains valuable repository/diagnostic state;
- the problem is insufficient reasoning depth rather than independence or isolation;
- no read-only or independent-review boundary would be weakened.

Create a fresh agent when independence, parallel ownership, clean review perspective, Git isolation, or a genuinely different role provides value.

# Other role routing

## Controller

Use the current main-session model by default.

For long ORCHESTRATED programs with extensive cross-system reasoning, a GPT-6 Astra Controller is preferred when available and cost is justified because preserving orchestration state can reduce repeated rediscovery.

Do not create a separate Controller subagent solely for model selection.

## Explorer

Use a lower-cost capable model for bounded ownership/dependency mapping.

- routine locate/map work -> Luna or Terra class;
- difficult architectural/ownership inference across a large codebase -> Astra medium/high when justified.

Explorer remains optional.

## Bug Investigator

- straightforward diagnosis -> lower-cost capable model;
- non-trivial cross-module/root-cause work -> Terra high/xhigh or Astra medium depending runtime availability and expected horizon;
- long-horizon multi-hypothesis, concurrency/lifetime/state corruption -> **Astra high/xhigh**;
- Critical/repeatedly unresolved -> Astra max only after boundary/invariant review.

The Investigator should resolve uncertainty before implementation when possible, but do not split diagnosis from implementation when one context can safely own both.

## Reviewer

Route by defect-finding difficulty, not by Developer model name.

- bounded ordinary review -> Luna/Terra class;
- High-risk security/concurrency/schema/protocol/public-behavior review -> Terra high/xhigh or Astra medium/high;
- long, broad, evidence-heavy review -> prefer Astra when continuity and cross-file reasoning dominate.

## Integration Reviewer

- bounded integrated change -> Terra xhigh or equivalent capable model;
- broad Cross-system integration, difficult conflict resolution, concurrency/security/data-migration interactions -> **Astra high/xhigh**;
- repeatedly unresolved Critical integration -> Astra max only when justified.

# Developer blocked-boundary rule

When a Developer discovers a consequential assumption is false, it should not silently redesign the solution or expand scope.

Return:

- expected assumption;
- actual repository evidence;
- affected/blocked slice;
- independent work that can safely continue, if any;
- why a decision is needed;
- smallest Controller correction required.

If the runtime supports asking while continuing unrelated work, use that capability for consequential questions rather than idling the whole task.

Routine local choices should be resolved from repository conventions without escalation.

# Context continuity

Long context is useful only when it preserves relevant state.

Prefer maintaining the same context for:

- one Developer's implementation plus compatible repair work;
- a long-lived Controller's orchestration state;
- difficult diagnosis where hypotheses/evidence accumulate over time.

Split contexts for:

- independent review;
- parallel ownership;
- Git isolation;
- unrelated tasks;
- deliberate fresh-perspective diagnosis.

Do not repeatedly paste full Explorer reports or history when stable boundaries/invariants plus searchable retained context are sufficient.

# Fallback when Astra is unavailable

Preserve the routing **intent**, not the exact model name.

For Astra-class tasks, choose the strongest available GPT-5.6 path appropriate to the same reasoning requirement, typically Terra high/xhigh/max or Sol for the hardest unresolved work.

Do not downgrade governance merely because the preferred model is unavailable.

# Cost and quality control

Prefer:

`make consequential boundary explicit -> choose cheapest capable model -> preserve useful context -> validate -> review proportional to risk -> escalate reasoning only when evidence justifies it`

not:

`unclear plan -> strongest model by default -> repeated handoffs -> larger diff -> repeated full validation`

Model routing is an optimization layer. Correctness, user requirements, project constraints, and safe Git behavior remain authoritative.
