# Bugfix Workflow

Use this reference for defects, regressions, crashes, hangs, incorrect behavior, data corruption, race conditions, resource leaks, compatibility failures, or other bug-fixing work.

The central rule is: **do not confuse the observed symptom with the root cause.** A Bugfix is complete only when the change is causally justified and the original failure is meaningfully verified against regression.

## 1. Capture the bug precisely

Record the most useful available facts:

- observed symptom;
- expected behavior;
- reproduction steps, if known;
- environment/configuration relevant to the failure;
- frequency: deterministic, intermittent, or unknown;
- logs, stack traces, crash dumps, failing tests, screenshots, traces, or other evidence;
- first known bad version/commit when available;
- known workarounds or conditions that suppress the failure.

Do not invent missing reproduction details.

## 2. Decide Simple vs Complex

A Bugfix may use the **Simple** path when:

- the defect is localized;
- the causal chain is clear from code or a deterministic failing test;
- the blast radius is small;
- validation is straightforward;
- no significant cross-module, concurrency, lifetime, state, protocol, schema, or compatibility uncertainty exists.

Treat the Bugfix as **Complex** when any of these are true:

- root cause is unknown or only hypothesized;
- reproduction is intermittent or environment-sensitive;
- multiple modules participate in the failure;
- concurrency, ownership, object lifetime, ordering, caching, state machines, persistence, protocol, or memory corruption may be involved;
- multiple plausible root causes exist;
- the fix changes shared/public behavior or has a large regression surface;
- the final code change may be small but diagnosis is non-trivial.

Do not use diff size as the primary complexity signal.

## 3. Investigation phase for Complex bugs

Use an exploration-only **Bug Investigator** subagent, preferably the Explorer model configuration. The investigator must not modify production code.

The investigation should produce:

1. **Symptom** — what is actually observed.
2. **Reproduction/Evidence** — deterministic repro when possible; otherwise the strongest available evidence.
3. **Relevant execution path** — call chain, state transitions, ownership/lifetime, threads, I/O, persistence, or protocol flow.
4. **Candidate root causes** — ordered by evidence, not intuition.
5. **Discriminating checks** — tests, instrumentation, logging, breakpoints, assertions, traces, or code-path checks that can separate candidates.
6. **Root cause conclusion** — the most defensible causal explanation, with confidence and evidence.
7. **Affected surface** — files, modules, interfaces, tests, hot files, compatibility risks.
8. **Recommended fix boundary** — the smallest safe correction.
9. **Regression strategy** — how to prove the defect is fixed and stays fixed.

If no root cause can be established, report that explicitly. The controller may authorize a mitigation, diagnostic instrumentation, or hypothesis-driven change, but it must not be mislabeled as a confirmed fix.

## 4. Root-cause gate

Before the final implementation is accepted, answer:

- Why does the current code produce the observed symptom?
- Why does the proposed change break that causal chain?
- Why is the change narrower or safer than alternative fixes?
- What evidence would falsify this root-cause explanation?

For obvious Simple bugs, this can be concise. For complex bugs, require concrete evidence such as a failing regression test, trace, invariant violation, lifetime/ownership proof, API contract violation, or deterministic code-path demonstration.

## 5. Implementation

The Developer should prefer the smallest fix that addresses the proven root cause without unrelated cleanup.

Avoid common failure modes:

- suppressing an error without fixing the invalid state that caused it;
- adding retries, sleeps, null checks, or broad exception handling only to hide the symptom;
- weakening assertions or tests so the failure disappears;
- changing unrelated architecture while diagnosing the bug;
- treating one successful manual run as proof of correctness.

A defensive guard may still be valid when it is part of the correct contract, but the Developer must explain whether it is the causal fix, an additional hardening measure, or only a mitigation.

## 6. Regression verification

Whenever practical, create or update a regression test that fails before the fix and passes after it.

Preferred evidence order:

1. automated regression test reproducing the original defect;
2. existing deterministic test extended to cover the failure condition;
3. reproducible integration/system test;
4. deterministic manual reproduction with documented before/after behavior;
5. code-path/invariant proof plus targeted validation when direct reproduction is infeasible.

For intermittent bugs, repeat the relevant stress/concurrency/reliability scenario enough to provide meaningful evidence. Do not claim mathematical certainty from a finite stress run.

If a regression test is impractical, state why and record the substitute verification.

## 7. Developer handoff for Bugfix

In addition to the normal task handoff, require:

- **Symptom**
- **Root Cause**
- **Evidence**
- **Fix**
- **Regression Verification**
- **Residual Risk**

Example structure:

```text
Symptom:
...

Root Cause:
...

Evidence:
...

Fix:
...

Regression Verification:
...

Residual Risk:
...
```

## 8. Review requirements

The independent Reviewer must verify not only code correctness, but also the causal claim:

- Does the stated root cause actually explain the reported symptom?
- Does the patch address the root cause rather than only suppress the symptom?
- Could the patch introduce a new failure mode?
- Is regression verification capable of detecting the original defect?
- Are important adjacent paths still covered?

If the root-cause argument is weak, report it as a material finding even if the patch appears plausible.

## 9. Completion states

Use precise completion language:

- **Confirmed fix** — root cause is supported and regression verification passes.
- **Mitigation** — impact is reduced but root cause is not fully eliminated or proven.
- **Diagnostic change** — instrumentation/evidence gathering was added; bug is not yet fixed.
- **Hypothesis-driven fix** — a plausible change was made but causal proof remains incomplete.

Only use **Confirmed fix** when the evidence justifies it.
