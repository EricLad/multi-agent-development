# Bugfix Workflow

Use bug-fixing rigor in proportion to diagnosis difficulty and risk.

## FAST bugfix

Use the same coding context to investigate and fix when the defect is localized, deterministic, and causally obvious.

Typical flow:

`symptom -> inspect relevant path -> identify cause -> fix -> targeted regression/build/test -> inspect diff`

Do not spawn a separate Bug Investigator or require a formal root-cause report for a trivial obvious defect.

Still avoid symptom-only patches when the actual cause is visible.

## STANDARD bugfix

Use one Developer to investigate + implement when the bug is non-trivial but still belongs to one coherent ownership surface.

The Developer should report concisely:

- symptom;
- root cause or best-supported explanation;
- fix;
- regression verification;
- meaningful residual uncertainty.

Use an independent Reviewer when risk/uncertainty warrants it.

## ORCHESTRATED / separate Bug Investigator

Use a separate Bug Investigator only when **diagnosis itself is a substantial independent problem**, for example:

- root cause unknown;
- intermittent/environment-sensitive reproduction;
- multiple modules or plausible causes;
- concurrency, async ordering, ownership/lifetime, state corruption, persistence, protocol, memory corruption;
- large blast radius or difficult validation.

The Investigator is read-only with respect to production code.

### Investigator output

Return only evidence useful to the Controller/Developer:

1. **Symptom**
2. **Reproduction/Evidence**
3. **Relevant execution path**
4. **Candidate root causes** ordered by evidence
5. **Discriminating checks**
6. **Root-cause conclusion** with confidence
7. **Affected/risky surfaces**
8. **Recommended fix boundary**
9. **Regression strategy**

Do not pad the handoff with implementation details the Developer can inspect directly.

## Root-cause principle

For non-trivial bugs, distinguish symptom from root cause.

Ask as appropriate:

- why does current code produce the symptom?;
- why does the change break that causal chain?;
- what evidence supports this explanation?;
- what regression check can detect the original failure?

If root cause cannot be established, say so. A mitigation or hypothesis-driven change must not be mislabeled as a confirmed fix.

## Regression verification

Prefer the cheapest evidence that is strong enough for the bug:

1. automated regression test;
2. deterministic existing/integration test;
3. documented reproducible before/after scenario;
4. targeted stress/concurrency run for intermittent defects;
5. code-path/invariant evidence when direct reproduction is impractical.

Do not demand an elaborate new test harness for a trivial bug when targeted existing validation is sufficient.

## Completion language for uncertain bugs

Use precise states when uncertainty matters:

- **Confirmed fix** — root cause supported and regression verification passes;
- **Mitigation** — impact reduced but cause not fully eliminated/proven;
- **Diagnostic change** — evidence gathering added; bug not fixed;
- **Hypothesis-driven fix** — plausible change but causal proof incomplete.

For obvious FAST bug fixes, concise plain-language completion is sufficient; these labels are mainly for non-trivial/uncertain cases.