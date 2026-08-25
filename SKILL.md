---
name: multi-agent-development
description: Orchestrate software feature development in Codex with a controller-only main session and specialized subagents for exploration, implementation, independent review, integration review, testing, and Git worktree-based parallel development. Use when implementing a new feature, making a non-trivial code change, coordinating multiple coding agents, deciding whether work should be split, using worktrees for parallel implementation, or enforcing independent code-review and integration quality gates.
---

# Multi-Agent Development

Use the main conversation as the **Controller / Tech Lead**. The controller owns requirements, decomposition, coordination, arbitration, integration, and final acceptance. It should not implement production code itself unless no subagent mechanism is available and the user explicitly permits a fallback.

## Core rules

1. Clarify material ambiguity before implementation. Do not ask about details that can be safely resolved from the repository or project instructions.
2. Read and obey the repository's `AGENTS.md`, build instructions, test instructions, architecture constraints, and relevant local documentation before assigning implementation work.
3. Classify the task as **Simple** or **Complex** based on coupling, affected modules, dependency structure, risk, and whether independent parallel work is possible. Do not classify by line count alone.
4. Keep roles independent: **Developer != Reviewer**. A developer never performs the authoritative review of its own change.
5. Every implementation task must pass its scoped build/tests before review.
6. Reviewer findings are evidence, not self-executing decisions. The developer may confirm or dispute them with evidence; the controller arbitrates unresolved material findings.
7. Parallelize only tasks with safe ownership boundaries. Avoid assigning the same hot file or shared API to multiple workers unless dependencies are explicitly ordered.
8. For complex parallel work, use isolated worktrees/branches. Merge only controller-approved work.
9. After all complex-task branches are integrated, run an independent **Integration Review** over the complete `BASE_COMMIT..FINAL_HEAD` change and run the project-appropriate full validation suite.
10. Clean up temporary worktrees/branches only after final acceptance, and never delete user-owned branches or uncommitted work.

## Model routing

When the exact configurations are available, prefer:

- Explorer: **5.6 Terra xhigh**
- Developer: **5.6 Terra xhigh**
- Reviewer: **5.6 Luna xhigh**, code-review mode
- Integration Reviewer: **5.6 Luna xhigh**, code-review mode

If an exact model or mode is unavailable, use the closest available capable model while preserving role separation and review independence. State the downgrade once; do not silently change the workflow.

## Workflow selection

### Simple task

Use this path when the change has a clear scope, low coupling, no meaningful parallel decomposition, and a small integration surface:

`requirements -> Developer -> scoped build/test -> independent Reviewer -> Developer fix/dispute -> controller arbitration/final review -> done`

A separate worktree is not required. Prefer a clear branch or commit boundary when practical so review and rollback remain easy.

Read:
- `references/task-contract.md` before assigning the Developer.
- `references/worker.md` for implementation rules.
- `references/reviewer.md` before independent review.
- `references/quality-and-git.md` for validation and Git safety.

### Complex task

Use this path when the change crosses modules, has uncertain impact, benefits from parallelism, changes shared interfaces/state/lifecycle, or carries meaningful regression risk:

`requirements -> Explorer -> dependency/impact analysis -> controller decomposition -> parallel Developers in isolated worktrees -> scoped build/test -> independent per-task Reviewers -> Developer fix/dispute -> controller arbitration -> merge -> Integration Reviewer -> clean/full validation -> controller final review -> cleanup`

Before decomposition, run an exploration-only subagent. The Explorer must not modify production code.

Read:
- `references/explorer.md` before exploration.
- `references/task-contract.md` before task assignment.
- `references/worker.md` before development.
- `references/reviewer.md` before per-task review.
- `references/integration-reviewer.md` before final integration review.
- `references/quality-and-git.md` before creating, merging, or deleting worktrees/branches.

## Controller responsibilities

The controller must:

- preserve the user's intent and acceptance criteria;
- maintain a task/dependency map for complex work;
- record the baseline commit for complex changes before implementation begins;
- prevent overlapping workers from editing shared hot files without an explicit plan;
- give every Developer a bounded task contract;
- require concrete build/test evidence rather than accepting "done" claims;
- route review findings back to the original Developer for repair or evidence-based dispute;
- arbitrate material Developer/Reviewer disagreements;
- review each task before merge;
- perform or delegate safe integration and resolve conflicts without silently changing task semantics;
- require final integration review and project-appropriate validation for complex work;
- report what changed, what was validated, material residual risks, and any intentionally deferred findings.

## User interaction

Do not overwhelm the user with internal orchestration details. Ask only when a material product/architecture choice cannot be safely inferred. For long-running multi-agent work, provide concise progress updates at meaningful milestones: exploration complete, decomposition decided, implementation/review status, integration result, and final validation.

## Failure handling

If a worker fails, stalls, edits outside its task boundary, or cannot validate its work, the controller should diagnose and reassign or narrow the task rather than absorbing implementation into the main session by default.

If parallel tasks turn out to have unsafe overlap, stop further parallel modification of the overlapping surface and serialize or refactor the dependency plan.

If the final integration review discovers a defect, route it to the most relevant original Developer when possible, then repeat the necessary review and validation gates.

## Completion criteria

Do not declare the feature complete until all applicable conditions hold:

- user requirements and acceptance criteria are satisfied;
- relevant project instructions were followed;
- scoped implementation builds/tests passed;
- blocking review findings are resolved;
- controller task-level review passed;
- complex changes are merged successfully;
- complex changes passed Integration Review;
- project-appropriate final build/tests/regression checks passed;
- temporary worktrees created by this workflow are safely cleaned up.
