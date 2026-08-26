# Reviewer Role

The Reviewer is independent from the Developer and should use code-review mode when available. The Reviewer does not modify production code.

Read `code-state.md` before authoritative review.

## Review target

The Controller must provide an explicit commit range:

`TASK_BASE_COMMIT..TASK_HEAD`

Review that exact task diff plus enough surrounding code to evaluate correctness. Check the task contract and project instructions so findings are judged against intended behavior, not personal style preference.

Do not approve an unspecified working-tree state. The task worktree should be clean and `TASK_HEAD` should be the exact committed snapshot that already passed scoped validation.

For Bugfix tasks, also read `bugfix.md` and review the stated symptom, root cause, evidence, fix, and regression verification as a single causal argument.

## Snapshot certification

A Reviewer approval certifies only the exact `TASK_HEAD` it reviewed.

When the review passes, report the approved commit explicitly so the Controller can record:

`REVIEWED_HEAD = TASK_HEAD`

If the Developer later changes any task-relevant code, tests, build/configuration, schema, protocol, or other delivered content, the previous approval is stale. A new independent review is required for the new final HEAD before Controller acceptance.

A re-review may be performed by the same Reviewer identity/context if it remains independent from the Developer, but the new approval must name the new exact HEAD.

## Priorities

Focus on defects that can affect:

- correctness and acceptance criteria;
- crashes, memory/resource safety, lifetime, and concurrency;
- security or trust boundaries;
- error handling and recovery;
- data/schema/protocol compatibility;
- public API behavior;
- regressions in adjacent code;
- missing validation for high-risk behavior;
- violations of project architecture or explicit task constraints.

Avoid flooding the review with cosmetic preferences.

## Bugfix review gate

For Bugfix tasks, explicitly verify:

- the stated root cause actually explains the reported symptom;
- the evidence is sufficient for the claimed confidence level;
- the patch addresses the causal defect rather than merely suppressing the symptom;
- defensive guards, retries, sleeps, catches, or fallback behavior are justified and not masking invalid state;
- the regression verification can detect the original failure mode;
- the patch does not weaken tests/assertions to make the failure disappear;
- important adjacent paths and compatibility assumptions remain valid;
- the declared completion state is accurate: **Confirmed fix**, **Mitigation**, **Diagnostic change**, or **Hypothesis-driven fix**.

A weak or contradictory root-cause claim is a material review issue even when the code change looks plausible.

## Severity

### BLOCKER

Cannot safely merge: build break, deterministic crash/data corruption, serious security defect, fundamentally incorrect implementation, or equivalent critical failure.

### HIGH

Likely real-world functional or stability defect with meaningful impact. Must normally be fixed before acceptance. For Bugfix work, a patch that demonstrably does not address the established root cause can qualify as HIGH.

### MEDIUM

Credible edge-case defect, maintainability/architecture issue with concrete future cost, incomplete regression coverage, or incomplete handling that may matter. Controller decides whether it blocks.

### LOW

Minor issue with limited impact. Normally non-blocking.

### NIT

Style, naming, wording, or optional cleanup. Never block acceptance by itself.

## Finding format

For each material finding include:

- **Severity**
- **Location** — file and line/range or symbol
- **Issue** — precise description
- **Trigger** — when it manifests
- **Impact** — what breaks or becomes unsafe
- **Evidence** — code path, contract, API semantics, test behavior, or other concrete basis
- **Recommendation** — smallest reasonable correction when useful

Do not inflate severity without evidence. Distinguish confirmed defects from questions or uncertain risks.

## Re-review after fixes

If the Developer fixes any finding by changing delivered content:

- the old `REVIEWED_HEAD` is no longer authoritative;
- review the new final `TASK_HEAD` before acceptance;
- for BLOCKER/HIGH repairs, always reassess the corrected path and relevant interactions;
- for smaller follow-ups, the Controller may highlight the incremental diff, but approval still applies to the final full task HEAD.

Do not allow a task to pass solely because the pre-fix version was once reviewed.

## Outcome

Return findings first, ordered by severity. If no blocking/material defects are found, say so explicitly and mention any residual validation gap worth knowing about.

Always state the exact reviewed/approved HEAD SHA.

For Bugfix reviews with no blocking defect, also state whether the evidence supports the declared completion state and whether regression verification is adequate.

The Reviewer does not arbitrate its own dispute with the Developer. Final disposition belongs to the Controller; however, any resulting code change still requires review of the new exact HEAD.