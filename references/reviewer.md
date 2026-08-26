# Reviewer Role

Use an independent Reviewer when the workflow tier/risk justifies independent review.

- FAST: no separate Reviewer by default; the main session performs final diff inspection.
- STANDARD: Reviewer is conditional.
- ORCHESTRATED: independent per-task review is a normal quality gate.

The Reviewer never modifies production code.

# STANDARD review

For STANDARD work, review the final implementation diff/state plus enough surrounding code to judge correctness.

A strict commit range is useful when available, but is not mandatory merely because the Skill is active.

Focus on material issues:

- acceptance criteria/correctness;
- regressions;
- error handling;
- security/trust boundaries;
- concurrency/lifetime/resource safety;
- persistence/data compatibility;
- public API/schema/protocol behavior;
- missing validation where risk is meaningful.

Avoid cosmetic review noise.

If the Developer changes code in response to a material finding, rerun relevant validation and re-review when the correction materially changes the reviewed behavior. Do not force another independent review for trivial non-semantic edits unless risk warrants it.

# ORCHESTRATED review

Read `code-state.md` before authoritative ORCHESTRATED review.

The Controller provides:

`TASK_BASE_COMMIT..TASK_HEAD`

Review that exact diff plus required surrounding code and task contract.

A passing review certifies only that exact `TASK_HEAD`:

`REVIEWED_HEAD = TASK_HEAD`

If delivered code/tests/build/config change afterward, the previous approval is stale and the final new HEAD must be independently reviewed before acceptance.

# Bugfix review

For a non-trivial Bugfix, review the causal argument as appropriate:

- does the stated root cause explain the symptom?;
- does the patch address the cause rather than only suppress the symptom?;
- can regression verification detect the original failure?;
- does the declared confidence/completion state match the evidence?;
- are retries/sleeps/guards/catches masking invalid state?

Obvious FAST bugs do not need a separate causal-review ceremony unless risk/uncertainty grows.

# Severity

## BLOCKER

Cannot safely accept: build break, deterministic crash/data corruption, serious security defect, fundamentally incorrect implementation, or equivalent critical failure.

## HIGH

Likely meaningful functional/stability/security/correctness defect. Normally blocks acceptance.

## MEDIUM

Credible edge case, incomplete handling/validation, or architecture issue with concrete impact. Controller decides whether it blocks.

## LOW

Minor issue with limited impact.

## NIT

Style/naming/wording/optional cleanup. Never blocks by itself.

# Finding format

For material findings include:

- **Severity**
- **Location**
- **Issue**
- **Trigger**
- **Impact**
- **Evidence**
- **Recommendation** when useful

Do not inflate severity or flood the review with preferences.

# Outcome

Return findings first, ordered by severity. If no material issue is found, say so explicitly and mention meaningful residual validation gaps.

For ORCHESTRATED review, always state the exact reviewed/approved HEAD SHA.

For STANDARD review, keep the output concise and proportional to the change.