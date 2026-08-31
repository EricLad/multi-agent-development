# Integration Reviewer Role

Use this role only for **ORCHESTRATED** work after accepted task snapshots have been integrated into the workflow staging branch.

FAST and STANDARD do not need an Integration Reviewer by default.

## Review range

Review:

`STAGING_BASE_COMMIT..STAGING_HEAD`

The goal is to find defects that appear only when independently implemented changes interact.

## Focus

Check especially for:

- incompatible assumptions between tasks;
- API/contract mismatch;
- duplicated or contradictory behavior;
- initialization/shutdown ordering;
- ownership/lifetime/concurrency interactions;
- state/data-flow inconsistencies;
- build-system integration errors;
- missing registrations/call sites/migrations/wiring;
- regressions outside individual task boundaries;
- merge-conflict semantic damage;
- isolated tests that no longer represent integrated behavior.

## Batch review rule

Perform a complete pass over the integrated staging snapshot before returning findings when practical.

- collect material BLOCKER/HIGH/MEDIUM findings into one batch;
- do not intentionally stop after the first non-BLOCKER issue;
- stop early only when a BLOCKER makes further review impossible or meaningless;
- distinguish current-contract defects from Optional Hardening/Deferred work using the dispositions in `reviewer.md`.

The goal is one integrated finding batch followed by one grouped repair/revalidation/re-review cycle when practical.

## Validation context

Use final staging build/test/integration results as evidence, but do not treat green tests as proof that integration is correct.

The staging validation should follow the Validation Pyramid from `quality-and-git.md`: expensive full/integration/real-data checks belong primarily at the staging gate rather than being redundantly repeated by every task.

## Findings

Use the same material severity, disposition, and finding format as `reviewer.md`.

For BLOCKER/HIGH findings, identify the most likely owning task/subsystem so repair can be routed efficiently.

When multiple findings can be fixed safely together, recommend a single batch repair rather than one repair cycle per finding.

## Re-review and convergence

After a batch integration repair:

1. inspect the exact new `STAGING_HEAD`;
2. verify the previous required findings are resolved;
3. inspect the combined repair for new integration regressions;
4. use refreshed staging validation evidence.

If a second material integration repair/re-review cycle still produces substantial new BLOCKER/HIGH/MEDIUM findings, report **CONVERGENCE_REQUIRED** to the Controller rather than assuming unlimited additional loops.

The Controller should re-check integration architecture, task boundaries, missing invariants, or whether review is expanding the accepted contract.

## Completion

Integration Review passes when no unresolved BLOCKER/HIGH Required Defect remains, Contract Gaps are resolved, and material MEDIUM findings are explicitly dispositioned.

Approval certifies only the exact `STAGING_HEAD` reviewed. A later staging code change invalidates that approval and requires re-review according to `code-state.md`.