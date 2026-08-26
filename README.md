# multi-agent-development

[中文](#中文) · [English](#english)

A multi-agent software development workflow for Codex.

> Keep the main Codex session as the **Controller / Tech Lead**. Delegate exploration, implementation, bug investigation, independent review, and integration review to specialized subagents, while binding every quality gate to exact Git commits.

---

# 中文

## 简介

`multi-agent-development` 是一个面向 **Codex** 的多代理软件开发 Skill。

核心原则是：**主会话不直接承担生产代码实现，而是作为 Controller / Tech Lead，负责需求确认、复杂度判断、代码探索调度、任务拆分、并发调度、审查裁决、集成、Git 生命周期和最终验收；实际编码交给子代理完成。**

适用于：

- Feature 开发；
- Bug 修复；
- Refactor；
- Performance 优化；
- Maintenance 和其他非平凡代码修改。

## 核心角色

| 角色 | 职责 | 优先模型配置 |
| --- | --- | --- |
| Controller / Tech Lead | 需求、拆分、调度、裁决、集成、最终验收 | 当前主会话 |
| Explorer / Bug Investigator | 只读探索、影响面/依赖/根因分析 | 5.6 Terra xhigh |
| Developer | 实现一个有明确边界的任务并完成 scoped validation | 5.6 Terra xhigh |
| Reviewer | 独立审查具体 task commit，不直接改代码 | 5.6 Luna xhigh + Code Review |
| Integration Reviewer | 审查最终 staging 的完整集成 diff | 5.6 Luna xhigh + Code Review |

如果当前 Codex 环境无法选择完全相同的模型或模式，Skill 会要求使用最接近的可用配置，同时保留角色隔离和独立审查原则。

## 最重要的工程原则：质量门认证的是 Commit，不是“任务”

每个实现任务都会维护：

```text
TASK_BASE_COMMIT
TASK_HEAD
VALIDATED_HEAD
REVIEWED_HEAD
ACCEPTED_HEAD
```

只有：

```text
TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD
```

这个 task 才允许进入集成阶段。

因此如果 Reviewer 发现问题，Developer 修改后产生了新的 commit：

```text
旧 Review approval → 自动失效
旧 Validation      → 如果代码已变化则自动失效
                    ↓
重新 Build/Test
                    ↓
重新 Independent Review
```

不会再出现“Reviewer 审的是旧代码，但最终合并的是修复后的新代码”的漏洞。

## Repository Preflight

在任何 Developer 开始修改代码之前，Controller 先记录：

```text
USER_TARGET_BRANCH
TARGET_BASE_COMMIT
当前 git status
已有 worktrees
已有相关 local branches
```

如果当前工作树有用户未提交修改，Skill 不会把自己的修改直接混进去。优先使用隔离 branch/worktree；如果新任务本身依赖这些未提交修改，则需要明确制定安全方案。

## Simple Task

简单任务不需要额外 worktree，但默认仍使用一个 workflow-owned 临时 branch，让用户目标分支在 Review 通过前保持干净。

```text
Preflight
    ↓
Temporary Task Branch
    ↓
Developer
    ↓
Commit All Changes
    ↓
Working Tree Clean
    ↓
Scoped Build/Test on TASK_HEAD
    ↓
Independent Reviewer
TASK_BASE_COMMIT..TASK_HEAD
    ↓
Fix? → New Commit → Revalidate → Re-review
    ↓
Controller accepts exact HEAD
    ↓
Promote to User Target
    ↓
Delete Temporary Branch
```

## Complex Task

复杂任务不会直接把各个 worktree 分支合并进用户目标分支。

Skill 会先创建一个 workflow-owned **staging branch**：

```text
USER_TARGET_BRANCH
       │
       └── TARGET_BASE_COMMIT
                 ↓
           STAGING_BRANCH
                 ↓
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Task A     Task B     Task C
   Worktree   Worktree   Worktree
       ↓         ↓         ↓
  Developer  Developer  Developer
       ↓         ↓         ↓
  Commit +   Commit +   Commit +
  Validate   Validate   Validate
       ↓         ↓         ↓
  Reviewer   Reviewer   Reviewer
       ↓         ↓         ↓
 ACCEPTED    ACCEPTED    ACCEPTED
   HEAD A      HEAD B      HEAD C
       └─────────┼─────────┘
                 ↓
      Merge into Staging
      in dependency order
                 ↓
       Clean Build / Full Test
                 ↓
     STAGING_VALIDATED_HEAD
                 ↓
       Integration Reviewer
                 ↓
      STAGING_REVIEWED_HEAD
                 ↓
      Controller Final Gate
                 ↓
       Promote to User Target
                 ↓
       Mandatory Git Cleanup
```

最终 staging 必须满足：

```text
STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD
```

### 为什么增加 staging branch

这样即使 A、B、C 各自已经 Review 通过，它们组合以后仍然可以先在隔离的 staging 中进行：

- Clean Build；
- Full Test；
- Integration/Regression Test；
- Integration Review；
- 修复和重新验证。

只有最终集成快照通过以后才进入用户目标分支，因此 `main` / `main-re` 等目标分支可以一直保持最后已知良好状态。

## Target Drift

如果并行开发过程中用户目标分支被其他工作推进了：

```text
TARGET_BASE_COMMIT
       ↓
用户目标分支出现新 commit
```

Skill 不会在最后一步直接在用户目标分支上解决冲突然后宣布完成。

正确流程是：

```text
Latest User Target
        ↓
Integrate into Staging
        ↓
Resolve conflicts in Staging
        ↓
New STAGING_HEAD
        ↓
Full Validation again
        ↓
Integration Review again
        ↓
Promote certified result
```

## Dependency-aware Worktrees

Explorer 除了分析 hot files，还必须输出依赖图和推荐集成顺序。

每个 task contract 会明确：

```text
BASE_REF
TASK_BASE_COMMIT
PREDECESSORS
INTEGRATION_TARGET
```

例如：

```text
Task A ─────→ Task C
Task B ─────→ Task D
Task A + B ─→ Task E
```

Task C 不能因为 Agent Pool 有空位就提前启动；它必须等 Task A 的必需输出已经接受，并从包含该 predecessor 的正确 base 开始。

所有 accepted task 也按 dependency/topological order 进入 staging。

## Agent Pool / 并发调度

Skill **不会把 Codex 的最大子代理数量写死成 3、4 或 6**。

Controller 维护：

```text
Active Agents
Ready Queue
Blocked Queue
```

当并发上限未知时，默认最多先启动 **3 个并发 Developer**，并为 Reviewer / Investigator / Repair 保留容量。运行环境明确支持更高并发且代码边界安全时才增加 wave size。

Review 使用流水线：

```text
Developer A
   ↓ committed + validated handoff
释放 Agent A slot
   ↓
Reviewer A

同时 Developer B / C 继续运行
```

Agent slot 和 Worktree 生命周期彼此独立。

## Bugfix Workflow

Bug 的复杂度按**定位和验证难度**判断，而不是最终 diff 行数。

### Simple Bugfix

```text
Symptom
  ↓
Developer
  ↓
Root Cause + Fix
  ↓
Commit + Regression Verification
  ↓
Independent Review
  ↓
Confirmed Fix
```

### Complex Bugfix

```text
Symptom / Evidence
       ↓
Bug Investigator
       ↓
Reproduction / Evidence
       ↓
Candidate Root Causes
       ↓
Discriminating Checks
       ↓
Root Cause Conclusion
       ↓
Developer Fix
       ↓
Regression Verification
       ↓
Independent Review
       ↓
Staging Integration Review when applicable
```

Developer 的 Bugfix 交付必须说明：

```text
Symptom
Root Cause
Evidence
Fix
Regression Verification
Residual Risk
```

完成状态严格区分：

- `Confirmed fix`
- `Mitigation`
- `Diagnostic change`
- `Hypothesis-driven fix`

## Review 严重级别

| 级别 | 默认处理 |
| --- | --- |
| BLOCKER | 必须修复 |
| HIGH | 必须修复 |
| MEDIUM | Controller 裁决 |
| LOW | 通常不阻塞 |
| NIT | 不阻塞 |

Developer 可以 `Confirmed` 或 `Disputed + Evidence`，但不能自行否决重要 Finding。

任何代码修复导致 HEAD 改变后，都必须重新获得针对新 HEAD 的独立 Review。

## Git Resource Ledger 与自动清理

Skill 会记录**所有自己创建的临时 Git 资源**，不再只跟踪 worktree：

```text
Task Branch
Task Worktree
Simple-task Branch
Staging Branch
```

最终每个资源只能是：

```text
Deleted
```

或者：

```text
Retained with reason
```

普通 merge 优先使用：

```bash
git branch -d <branch>
```

如果明确使用 squash/rebase/cherry-pick 等不保留 ancestry 的集成方式，只有在 branch 确认由本 workflow 创建、accepted change 已进入最终认证结果、没有独有未集成修改等安全证明全部成立时，才允许受控使用：

```bash
git branch -D <branch>
```

不会为了“看起来干净”而强删不确定的用户代码。

## 安装

用户级 Codex Skill 默认位于 `$CODEX_HOME/skills`；未设置时通常为 `~/.codex/skills`。

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

安装后如果 Codex 没有立即发现 Skill，请重新启动 Codex。

## 更新

```bash
git pull
```

## 使用

开发功能：

```text
$multi-agent-development
给用户管理模块增加批量禁用功能。
```

修复 Bug：

```text
$multi-agent-development
程序偶尔在关闭用户会话时崩溃，请定位根因并修复。
```

## 项目结构

```text
multi-agent-development/
├── README.md
├── LICENSE
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── agent-pool.md
    ├── bugfix.md
    ├── code-state.md
    ├── explorer.md
    ├── integration-reviewer.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

## 与项目 `AGENTS.md` 的关系

这个 Skill 定义**通用软件工程工作流**；项目自己的 `AGENTS.md` 定义**项目级约束**。

两者叠加使用。

## License

MIT License. See [LICENSE](./LICENSE).

---

# English

## Overview

`multi-agent-development` is a multi-agent software-engineering Skill for **Codex**.

The main session acts as the **Controller / Tech Lead** while implementation, exploration, bug investigation, independent code review, and integration review are delegated to specialized subagents.

The workflow's defining property is **commit-bound certification**: tests and reviews certify exact Git snapshots, not vague task states.

## Per-task state

Every implementation task tracks:

```text
TASK_BASE_COMMIT
TASK_HEAD
VALIDATED_HEAD
REVIEWED_HEAD
ACCEPTED_HEAD
```

A task may be integrated only when:

```text
TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD
```

Any delivered-code change after validation or review invalidates stale certification and requires revalidation/re-review of the new final HEAD.

## Simple workflow

```text
preflight
 -> workflow-owned temporary branch
 -> Developer
 -> commit all changes
 -> clean working tree
 -> scoped validation on TASK_HEAD
 -> independent review of TASK_BASE_COMMIT..TASK_HEAD
 -> repair -> revalidate -> re-review when HEAD changes
 -> Controller accepts exact HEAD
 -> promote to user target
 -> cleanup temporary branch
```

## Complex workflow

Complex work uses a workflow-owned **staging branch** so the user's target branch remains at its last known good state until the integrated result is certified.

```text
preflight
 -> Explorer / dependency graph
 -> staging branch
 -> dependency-aware task worktrees
 -> Developers
 -> committed + validated task HEADs
 -> independent per-task Reviews
 -> accepted task HEADs
 -> integrate into staging in topological order
 -> clean/full/integration validation
 -> Integration Review
 -> repeat both gates if staging changes
 -> reconcile user-target drift in staging if necessary
 -> promote certified staging snapshot
 -> mandatory cleanup
```

Final staging must satisfy:

```text
STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD
```

## Dynamic Agent Pool

The Skill does not hardcode a universal Codex concurrency limit. It maintains Active, Ready, and Blocked activities, schedules in waves, reserves practical capacity for review/repair, and treats thread-limit errors as scheduler events.

## Bugfix

Bug complexity is based on diagnosis and validation difficulty, not diff size. Complex defects use a read-only Bug Investigator, root-cause evidence, regression verification, and independent review.

Completion states are intentionally precise: **Confirmed fix**, **Mitigation**, **Diagnostic change**, or **Hypothesis-driven fix**.

## Git safety

The workflow records every temporary branch/worktree it creates, including Simple-task branches and the Complex staging branch. Cleanup is mandatory after final acceptance, but user-owned or uncertain Git state is never force-deleted automatically.

## Installation

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

## Usage

```text
$multi-agent-development Implement this feature: ...
```

or:

```text
$multi-agent-development Investigate and fix this bug: ...
```

## License

MIT License. See [LICENSE](./LICENSE).