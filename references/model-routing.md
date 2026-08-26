# Role + Risk Adaptive Model Routing

Apply model routing only when subagents are actually used. Do not create subagents or bookkeeping solely to exercise this policy.

# Workflow-tier interaction

## FAST

No delegated-role routing by default. Use the current main-session model and keep the task in one coding context.

Do not record `ROLE/RISK_LEVEL/ASSIGNED_MODEL/...` fields for routine FAST work.

## STANDARD

Use one Developer when delegation is useful.

Suggested routing:

- bounded Low-risk implementation -> **Luna max** when available;
- normal production implementation -> **Terra xhigh**;
- High-risk bounded implementation -> **Terra max** when justified.

A STANDARD Reviewer is conditional. When used:

- routine review -> **Luna max**;
- High-risk/uncertain review -> **Terra xhigh**.

Keep routing notes concise. Full model-audit fields are unnecessary unless an escalation materially matters.

## ORCHESTRATED

Use the full adaptive routing below.

# Capability tiers

- **Luna max** — cost-efficient high-volume exploration/routine review/bounded low-risk implementation.
- **Terra xhigh** — default normal production implementation and stronger analysis tier.
- **Terra max** — high-risk implementation, difficult root-cause analysis, concurrency/lifetime/state-machine work.
- **Sol xhigh/max** — Critical or repeatedly unresolved work where stronger capability is materially valuable.

If an exact model/reasoning effort is unavailable, use the closest capable tier and state material downgrades once.

# Risk signals

Risk is independent from workflow structure and diff size.

## Low

Localized, reversible, small blast radius, deterministic validation, no material security/concurrency/lifetime/persistence/schema/protocol/public-compatibility concern.

## Medium

Normal production feature/refactor work with moderate reasoning/regression surface.

## High

Strong signals include:

- concurrency, async ordering, ownership/lifetime/resource safety;
- security/trust boundaries;
- persistent data/schema/migration correctness;
- protocol/serialization/public API compatibility;
- broad cross-module/state-machine behavior;
- difficult intermittent bugs;
- large blast radius or expensive rollback.

## Critical

Severe data/security impact, unusually difficult architecture, or failure of lower tiers to converge on a defensible solution.

# Role routing for ORCHESTRATED work

## Explorer

Default: **Luna max**.

Escalate to Terra xhigh only when the orchestration map itself needs difficult architectural/ownership inference.

Remember: Explorer is optional and should exist only when decomposition/orchestration needs a global map.

## Bug Investigator

- straightforward diagnosis -> Luna max;
- non-trivial cross-module/root-cause work -> Terra xhigh;
- concurrency/lifetime/state corruption or stubborn multi-hypothesis diagnosis -> Terra max;
- Critical/repeatedly unresolved -> Sol xhigh/max.

## Developer

- Low-risk/mechanical -> Luna max;
- normal production -> Terra xhigh;
- High-risk -> Terra max;
- Critical -> Sol xhigh/max when justified.

## Reviewer

Default: Luna max.

Escalate to Terra xhigh for High/Critical risk, material uncertainty, security/concurrency/lifetime/schema/protocol/broad public behavior, or difficult disputes.

## Integration Reviewer

Default: Terra xhigh.

Escalate to Terra max for High-risk integrated changes, meaningful conflict resolution, broad coupling, concurrency/security/data-migration concerns. Sol is reserved for Critical/unresolved integration uncertainty.

# Escalation triggers

Escalate when stronger capability is likely to improve correctness because:

- diagnosis/reasoning fails to converge;
- multiple plausible causes remain unresolved;
- repeated implementation/test cycles fail without progress;
- unexpected cross-module/hot-file coupling appears;
- a Reviewer reports architecture-level uncertainty;
- an evidence-based Developer/Reviewer dispute cannot be resolved confidently;
- validation exposes intermittent/non-local failures;
- merge/target-drift resolution materially increases complexity;
- actual risk is higher than initially believed.

Do not escalate merely because the task is long.

# Cost control

Use a stronger model only for the phase that needs it. Once uncertainty is resolved, unrelated low-risk work may return to its normal tier.

Do not sacrifice Developer/Reviewer independence when the workflow requires independent review.

# ORCHESTRATED audit fields

For ORCHESTRATED delegated activities, record when useful:

- `ROLE`;
- `RISK_LEVEL`;
- `ASSIGNED_MODEL`;
- `REASONING_EFFORT`;
- `MODEL_RATIONALE` when deviating from default;
- `ESCALATION_REASON` when upgraded.

These fields are not a completion ceremony for FAST/STANDARD.