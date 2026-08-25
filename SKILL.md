---
name: multi-agent-development
description: Orchestrate software development in Codex with a controller-only main session and specialized subagents for exploration, implementation, independent review, integration review, testing, bug diagnosis, regression verification, and Git worktree-based parallel development. Use when implementing features, fixing bugs, refactoring, optimizing performance, making non-trivial code changes, coordinating multiple coding agents, deciding whether work should be split, using worktrees for parallel implementation, or enforcing independent code-review and integration quality gates.
---

# Multi-Agent Development

Use the main conversation as the **Controller / Tech Lead**. The controller owns requirements, decomposition, coordination, arbitration, integration, and final acceptance. It should not implement production code itself unless no subagent mechanism is available and the user explicitly permits a fallback.

## Core rules

1. Clarify material ambiguity before implementation. Do not ask about details that can be safely resolved from the repository or project instructions.
2. Read and obey the repository's `AGENTS.md`, build instructions, test instructions, architecture constraints, and relevant local documentation before assigning implementation work.
3. Classify the request type as **Feature**, **Bugfix**, **Refactor**, **Performance**, or **Maintenance**, then classify execution complexity as **Simple** or **Complex**.
4. Complexity is based on diagnosis difficulty, coupling, affected modules, dependency structure, risk, and whether independent parallel work is possible. Do not classify by line count alone.
5. For Bugfix work, establish a defensible root cause before treating a code change as the final fix. Prefer reproduction or other concrete evidence, and require regression verification whenever practical.
6. Keep roles independent: **Developer != Reviewer**. A developer never performs the authoritative review of its own change.
7. Every implementation task must pass its scoped build/tests before review.
8. Reviewer findings are evidence, not self-executing decisions. The developer may confirm or dispute them with evidence; the controller arbitrates unresolved material findings.
9. Parallelize only tasks with safe ownership boundaries. Avoid assigning the same hot file or shared API to multiple workers unless dependencies are explicitly ordered.
10. For complex parallel work, use isolated worktrees/branches. Merge only controller-approved work.
11. After all complex-task branches are integrated, run an independent **Integration Review** over the complete `BASE_COMMIT..FINAL_HEAD` change and run the project-appropriate full validation suite.
12. Clean up temporary worktrees/branches only after final acceptance, and never delete user-owned branches or uncommitted work.

## Model routing

When the exact configurations are available, prefer:

- Explorer / Bug Investigator: **5.6 Terra xhigh**
- Developer: **5.6 Terra xhigh**
- Reviewer: **5.6 Luna xhigh**, code-review mode
- Integration Reviewer: **5.6 Luna xhigh**, code-review mode

If an exact model or mode is unavailable, use the closest available capable model while preserving role separation and review independence. State the downgrade once; do not silently change the workflow.

## Request-type routing

### Feature / Refactor / Performance / Maintenance

Use the normal Simple/Complex workflow below.

### Bugfix

Read `references/bugfix.md` before diagnosis or implementation.

The controller should establish, as applicable:

`reported symptom -> reproduction/evidence -> root cause -> bounded fix -> regression verification -> independent review`

A small diff can still be **Complex** if the bug is intermittent, root cause is unknown, crosses modules, involves concurrency/lifetime/state corruption, has a large blast radius, or is difficult to validate.

An obvious localized defect with a clear causal chain may use the Simple path without a separate exploration phase, but the Developer must still explain the root cause and regression verification.

Do not accept a speculative symptom patch as a completed Bugfix merely because the observed failure disappears once.

## Workflow selection

### Simple task

Use this path when the change has a clear scope, low coupling, no meaningful parallel decomposition, and a small integration surface. For Bugfix, the causal chain must also be sufficiently clear that a separate investigation phase is unnecessary.

`requirements/bug evidence -> Developer -> scoped build/test/regression verification -> independent Reviewer -> Developer fix/dispute -> controller arbitration/final review -> done`

A separate worktree is not required. Prefer a clear branch or commit boundary when practical so review and rollback remain easy.

Read:
- `references/bugfix.md` for Bugfix tasks.
- `references/task-contract.md` before assigning the Developer.
- `references/worker.md` for implementation rules.
- `references/reviewer.md` before independent review.
- `references/quality-and-git.md` for validation and Git safety.

### Complex task

Use this path when the change crosses modules, has uncertain impact, benefits from parallelism, changes shared interfaces/state/lifecycle, carries meaningful regression risk, or requires non-trivial bug diagnosis:

`requirements/evidence -> Explorer or Bug Investigator -> dependency/impact/root-cause analysis -> controller decomposition -> parallel Developers in isolated worktrees when safe -> scoped build/test -> independent per-task Reviewers -> Developer fix/dispute -> controller arbitration -> merge -> Integration Reviewer -> clean/full validation -> controller final review -> cleanup`

Before decomposition, run an exploration-only subagent. For Bugfix tasks, this agent acts as a **Bug Investigator** and must focus on reproduction evidence, causal chain, candidate root causes, and discriminating tests. It must not modify production code.

Read:
- `references/bugfix.md` for Bugfix tasks.
- `references/explorer.md` before exploration/investigation.
- `references/task-contract.md` before task assignment.
- `references/worker.md` before development.
- `references/reviewer.md` before per-task review.
- `references/integration-reviewer.md` before final integration review.
- `references/quality-and-git.md` before creating, merging, or deleting worktrees/branches.

## Controller responsibilities

The controller must:

- preserve the user's intent and acceptance criteria;
- classify the request type and execution complexity;
- for Bugfix, distinguish observed symptoms from proven root cause and avoid premature fixes;
- maintain a task/dependency map for complex work;
- record the baseline commit for complex changes before implementation begins;
- prevent overlapping workers from editing shared hot files without an explicit plan;
- give every Developer a bounded task contract;
- require concrete build/test evidence rather than accepting "done" claims;
- for Bugfix, require a root-cause statement and regression verification evidence before acceptance;
- route review findings back to the original Developer for repair or evidence-based dispute;
- arbitrate material Developer/Reviewer disagreements;
- review each task before merge;
- perform or delegate safe integration and resolve conflicts without silently changing task semantics;
- require final integration review and project-appropriate validation for complex work;
- report what changed, what was validated, material residual risks, and any intentionally deferred findings.

## User interaction

Do not overwhelm the user with internal orchestration details. Ask only when a material product/architecture choice cannot be safely inferred. For long-running multi-agent work, provide concise progress updates at meaningful milestones: exploration/investigation complete, decomposition decided, implementation/review status, integration result, and final validation.

For Bugfix tasks, clearly distinguish facts from hypotheses. If the root cause remains uncertain, say so rather than presenting a candidate fix as proven.

## Failure handling

If a worker fails, stalls, edits outside its task boundary, or cannot validate its work, the controller should diagnose and reassign or narrow the task rather than absorbing implementation into the main session by default.

If parallel tasks turn out to have unsafe overlap, stop further parallel modification of the overlapping surface and serialize or refactor the dependency plan.

If a Bugfix cannot be reproduced, gather alternative evidence such as logs, traces, invariants, failing tests, crash dumps, or a deterministic code-path proof. Do not fabricate reproduction. If the root cause remains unproven, classify the result as mitigation or hypothesis-driven change rather than a confirmed fix.

If the final integration review discovers a defect, route it to the most relevant original Developer when possible, then repeat the necessary review and validation gates.

## Completion criteria

Do not declare the work complete until all applicable conditions hold:

- user requirements and acceptance criteria are satisfied;
- relevant project instructions were followed;
- scoped implementation builds/tests passed;
- for Bugfix, root cause is documented with supporting evidence or the result is explicitly labeled as mitigation when proof is unavailable;
- for Bugfix, the original failure is covered by regression verification whenever practical;
- blocking review findings are resolved;
- controller task-level review passed;
- complex changes are merged successfully;
- complex changes passed Integration Review;
- project-appropriate final build/tests/regression checks passed;
- temporary worktrees created by this workflow are safely cleaned up.
