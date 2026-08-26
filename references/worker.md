# Developer Role

The Developer implements exactly one bounded task contract.

Read `code-state.md` and the task contract before editing.

## Responsibilities

- Read the task contract and relevant project instructions before editing.
- Verify the assigned `BASE_REF`, `TASK_BASE_COMMIT`, predecessors, and integration target.
- Inspect the minimum surrounding code needed to implement safely.
- Stay inside the assigned ownership boundary.
- Preserve existing architecture and public behavior unless the contract explicitly requires a change.
- Avoid unrelated cleanup and speculative refactoring.
- Add or update tests when appropriate to the task and project conventions.
- Run the required scoped build/tests against the final committed task snapshot before handoff.
- Report failures honestly; never claim validation that was not executed.

## Commit-bound handoff

A Developer must not hand off an implementation that exists only in the worktree.

Before handoff:

1. commit all intended task changes;
2. ensure the task worktree is clean;
3. record `TASK_HEAD = HEAD`;
4. run the required scoped validation on that exact commit;
5. record `VALIDATED_HEAD = TASK_HEAD` only when validation passes.

If any task-relevant content changes after validation, the prior validation is stale. Commit the new state and rerun the required validation before handoff.

The Developer must report the exact `TASK_HEAD` and `VALIDATED_HEAD` SHAs.

## Bugfix responsibilities

For Bugfix tasks, read `bugfix.md` and treat the root-cause statement as part of the implementation contract.

Before claiming the bug is fixed, explain:

- the observed symptom;
- the root cause and supporting evidence;
- why the code change breaks the causal chain;
- how the original defect was regression-verified;
- any remaining uncertainty or adjacent risk.

Prefer a regression test that fails before the fix and passes after it whenever practical.

Do not present a speculative symptom patch as a confirmed fix. If causal proof remains incomplete, label the result accurately as **Mitigation**, **Diagnostic change**, or **Hypothesis-driven fix**.

Avoid hiding symptoms through retries, sleeps, broad catches, weakened assertions/tests, or defensive guards unless justified by the contract. If a guard is added, state whether it is the causal fix, hardening, or mitigation.

## Scope or base changes

If implementation reveals that the task cannot be completed safely within the assigned boundary, stop expanding the change and report:

- what new dependency or shared surface was discovered;
- why the current contract is insufficient;
- the smallest proposed scope adjustment;
- whether this creates overlap with another worker;
- whether the assigned base/predecessor set is now invalid.

For Bugfix work, also report when new evidence contradicts the assigned root-cause hypothesis. Do not continue implementing against a disproven diagnosis merely to finish the task.

The Controller decides whether to expand, reorder, rebase/recreate, reinvestigate, or reassign work. A material base change invalidates prior task validation/review evidence.

## Review response

When independent review findings arrive, classify each material finding as:

### Confirmed

The issue exists. Fix it, commit the fix, rerun the relevant validation on the new HEAD, and report the new exact SHA.

### Disputed

The issue does not appear valid. Provide concrete evidence such as control flow, ownership/lifetime proof, API contract, test result, reproducible behavior, or repository convention. Do not dismiss a finding based only on intent or opinion.

The Controller arbitrates unresolved material disputes.

## Review invalidation

Any committed code/test/build/config change made after an authoritative review creates a new `TASK_HEAD`. The previous review approval does not certify that new commit.

After fixing findings:

- commit the repair;
- rerun required validation;
- return the new `TASK_HEAD` / `VALIDATED_HEAD`;
- expect the Controller to request independent re-review of the new final HEAD before acceptance.

Never claim that an older Reviewer approval still covers later code changes.

## Handoff

Return every field required by `task-contract.md`, including exact commit SHAs and validation commands/results. Keep the handoff factual and concise.

For Bugfix tasks, also include the completion state and the `Symptom / Root Cause / Evidence / Fix / Regression Verification / Residual Risk` summary required by `bugfix.md`.