# multi-agent-development

[中文](#中文) · [English](#english)

A multi-agent software development workflow for Codex with commit-bound quality gates, dynamic agent scheduling, staging integration, and risk-adaptive model routing.

---

# 中文

## 简介

`multi-agent-development` 是一个面向 **Codex** 的多代理软件开发 Skill。

核心原则是：**主会话作为 Controller / Tech Lead，负责需求、风险判断、任务拆分、模型路由、Agent 调度、Review 裁决、Git 集成和最终验收；生产代码交给独立子代理实现。**

它适用于：

- Feature 开发；
- Bug 修复；
- Refactor；
- Performance 优化；
- Maintenance；
- 多 worktree 并行开发；
- 需要独立 Review、Integration Review 和严格 Git 生命周期控制的代码修改。

## 核心角色

| 角色 | 职责 | 默认模型路由 |
| --- | --- | --- |
| Controller / Tech Lead | 需求、复杂度/风险判断、调度、裁决、集成、最终验收 | 当前主会话 |
| Explorer | 只读探索、调用链、依赖、hot files、验证地图 | **Luna max** |
| Bug Investigator | 根因调查、证据、候选原因、判别测试 | Luna max 起步，复杂时 Terra xhigh/max |
| Developer | 在 Task Contract 内实现并验证生产代码 | **Terra xhigh 默认**；低风险可 Luna max；高风险 Terra max |
| Reviewer | 独立审查单个 Task 的精确 commit diff | **Luna max**；高风险/争议升级 Terra xhigh |
| Integration Reviewer | 审查 staging 上完整集成结果 | **Terra xhigh**；高风险升级 Terra max |
| Critical Escalation | 极高风险或长期无法收敛的困难任务 | **Sol xhigh/max** |

模型不存在或 reasoning effort 不可选时，Skill 会要求使用最接近的可用能力层级，同时保留角色独立性和质量门。

## Role + Risk Adaptive Model Routing

这个 Skill **不再把某个角色永远固定到同一个模型**。

模型选择由两个维度决定：

```text
Role
+
Risk Level
↓
Assigned Model
```

风险等级独立于 Simple / Complex：

```text
Low
Medium
High
Critical
```

例如：

- 一个只有 1 行的鉴权修改，结构上可能是 Simple，但风险仍然可以是 High；
- 一个 Complex 任务中的普通代码搜索/依赖整理，仍然可以使用 Luna max；
- 一个普通生产功能 Developer 默认使用 Terra xhigh；
- 并发、生命周期、持久化、协议、安全等高风险修改自动倾向 Terra max。

### 默认路由

```text
Explorer
→ Luna max

Bug Investigator
→ Luna max
→ Terra xhigh
→ Terra max
→ Sol xhigh/max（Critical / 长期不收敛）

Developer
→ Luna max   （Low-risk / bounded / mechanical）
→ Terra xhigh（默认生产代码）
→ Terra max  （High-risk）
→ Sol        （Critical）

Reviewer
→ Luna max
→ Terra xhigh（High/Critical / 重大争议）

Integration Reviewer
→ Terra xhigh
→ Terra max  （High-risk integration）
→ Sol        （Critical / unresolved）
```

### 自动升级条件

当出现以下情况时，Controller 会考虑升级一个能力层级：

- Root Cause 长时间无法建立；
- 多个候选原因无法有效区分；
- 多轮实现/Test 失败但没有收敛；
- 出现意外跨模块耦合或 hot-file 冲突；
- Reviewer 发现架构级不确定性；
- Developer / Reviewer 出现无法可靠裁决的重要争议；
- 验证出现偶发或非局部失败；
- merge / target drift 让最终集成状态显著复杂化；
- 实际风险高于初始分类。

升级是质量控制机制，不是因为任务“写得长”。

## Simple Task

```text
Preflight
    ↓
Risk / Model Assignment
    ↓
Temporary Task Branch
    ↓
Developer
    ↓
Commit All Changes
    ↓
Working Tree Clean
    ↓
Scoped Validation on TASK_HEAD
    ↓
Independent Reviewer
    ↓
Repair / Dispute
    ↓
Revalidate + Re-review if HEAD changed
    ↓
Controller Accepts Exact HEAD
    ↓
Promote to User Target
    ↓
Cleanup
```

只有满足：

```text
TASK_HEAD
== VALIDATED_HEAD
== REVIEWED_HEAD
== ACCEPTED_HEAD
```

才允许进入目标分支。

## Complex Task

```text
Preflight
    ↓
Explorer / Bug Investigator
    ↓
Dependency + Risk + Hot-file Map
    ↓
Workflow Staging Branch
    ↓
Agent Pool / Concurrency Gate
    ↓
Task Developers in Worktrees
    ↓
Commit-bound Validation
    ↓
Independent per-task Reviewers
    ↓
Repair / Re-review / Model Escalation
    ↓
Controller Accepts Exact Task HEADs
    ↓
Merge ACCEPTED_HEADs into Staging
in dependency/topological order
    ↓
Clean Build / Full Tests
    ↓
Integration / Regression Checks
    ↓
Integration Reviewer
    ↓
Repair + Re-certification if needed
    ↓
Target Drift Reconciliation
    ↓
Promote Certified Staging to User Target
    ↓
Mandatory Cleanup
```

Complex 模式不会在中途把未经最终集成验证的 Task 直接合入用户目标分支。

最终必须：

```text
STAGING_HEAD
== STAGING_VALIDATED_HEAD
== STAGING_REVIEWED_HEAD
```

## Commit-bound Quality Gates

这个 Skill 的 Build/Test/Review 不是认证“Task”，而是认证**具体 commit**。

每个 Task 维护：

```text
TASK_BASE_COMMIT
TASK_HEAD
VALIDATED_HEAD
REVIEWED_HEAD
ACCEPTED_HEAD
```

如果 Reviewer 审的是 A，但 Developer 修复后产生 B：

```text
REVIEWED_HEAD = A
TASK_HEAD = B
```

则旧 Review 自动失效，B 必须重新 Validation + Review。

同样，如果代码在 Test PASS 后再次修改，旧 Validation 也自动失效。

## Bugfix Workflow

Bug 修复要求区分：

```text
Symptom
Root Cause
Evidence
Fix
Regression Verification
Residual Risk
```

对于复杂 Bug：

```text
Bug Symptom / Evidence
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
```

最终结果必须准确声明为：

- `Confirmed fix`
- `Mitigation`
- `Diagnostic change`
- `Hypothesis-driven fix`

不能因为“症状暂时没出现”就自动声明 Bug 已确认修复。

## Agent Pool / 并发调度

Skill 不把 Codex 最大子代理数写死成 3、4 或 6。

Controller 维护：

```text
Active Agents
Ready Queue
Blocked Queue
```

当并发上限未知时，默认先使用最多 **3 个并发 implementation Developers**，同时保留 Reviewer / Investigator / Repair 的容量；实际环境证明支持更多且任务能安全并行后，再谨慎扩容。

Agent slot 与 Worktree 生命周期分离：Developer 完成后可以释放 agent slot，但对应 branch/worktree 必须保留到 Review、修复、集成和最终 Cleanup 完成。

## Git / Worktree Safety

Complex 模式使用 workflow-owned staging branch，用户目标分支保持 last-known-good，直到最终 staging 通过完整认证。

Skill 维护统一 **Git Resource Ledger**，记录：

- Task branch；
- Worktree；
- Simple 临时 branch；
- Staging branch；
- ownership；
- base / predecessor / integration target；
- lifecycle state。

最终每个 workflow-created Git resource 必须：

```text
Deleted
```

或：

```text
Retained with reason
```

不会只删除 worktree 而静默遗留 task branch。

## 安装

Codex 用户级 Skill 默认位于 `$CODEX_HOME/skills`；未设置 `CODEX_HOME` 时通常是 `~/.codex/skills`。

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

设置了 `CODEX_HOME`：

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:CODEX_HOME\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

安装后如果未立即发现 Skill，请重新启动 Codex。

## 更新

```bash
git pull
```

## 使用

开发功能：

```text
$multi-agent-development
给用户管理模块增加批量禁用功能。按风险选择合适模型，复杂任务使用 worktree、独立 Review、staging 和最终 Integration Review。
```

修复 Bug：

```text
$multi-agent-development
程序偶尔在关闭用户会话时崩溃。先判断复杂度和风险；如果根因不明确，先调查 Root Cause，必要时升级模型，再进行修复、Regression Verification 和独立 Review。
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
    ├── model-routing.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

其中 `model-routing.md` 专门定义 Role + Risk 模型路由、风险等级以及自动升级规则；`code-state.md` 定义 commit-bound 状态机；`quality-and-git.md` 定义 staging、validation、promotion 和 cleanup。

## 与项目 `AGENTS.md` 的关系

`multi-agent-development` 定义通用的多代理工程流程；项目自己的 `AGENTS.md` 继续作为项目级语言、构建、测试和架构约束。

## License

MIT License. See [LICENSE](./LICENSE).

---

# English

## Overview

`multi-agent-development` is a multi-agent software development Skill for **Codex**. The main session acts as the Controller / Tech Lead while specialized subagents perform exploration, implementation, bug investigation, independent review, and integration review.

The workflow combines:

- role + risk adaptive model routing;
- dynamic Agent Pool scheduling;
- commit-bound validation and review;
- independent Developer / Reviewer contexts;
- root-cause-driven bug fixing;
- worktree-based parallel development;
- staging integration before the user target branch;
- mandatory Git cleanup.

## Adaptive model routing

Default routing when available:

| Role | Default / escalation |
| --- | --- |
| Explorer | Luna max |
| Bug Investigator | Luna max -> Terra xhigh -> Terra max -> Sol for Critical/unresolved work |
| Developer | Luna max for Low-risk bounded work; Terra xhigh by default; Terra max for High risk; Sol for Critical work |
| Reviewer | Luna max; Terra xhigh for High/Critical risk or material uncertainty |
| Integration Reviewer | Terra xhigh; Terra max for High-risk integration; Sol for Critical/unresolved integration |

Risk is independent from Simple/Complex classification. A tiny security, lifetime, persistence, or protocol change can still require a high-tier model.

## Commit-bound acceptance

Per task:

```text
TASK_HEAD == VALIDATED_HEAD == REVIEWED_HEAD == ACCEPTED_HEAD
```

Complex staging:

```text
STAGING_HEAD == STAGING_VALIDATED_HEAD == STAGING_REVIEWED_HEAD
```

Any delivered-code change invalidates stale validation and review certification.

## Complex workflow

```text
preflight
-> exploration / investigation
-> dependency + risk map
-> workflow staging branch
-> queued worktree Developers
-> commit-bound validation
-> independent per-task Reviewers
-> repair / re-review / escalation
-> integrate accepted commits into staging
-> full validation
-> Integration Reviewer
-> target-drift reconciliation
-> promote certified staging snapshot
-> cleanup
```

## Installation

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

Restart Codex if the new Skill is not discovered immediately.

## Repository structure

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
    ├── model-routing.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

## License

MIT License. See [LICENSE](./LICENSE).
