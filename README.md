# multi-agent-development

[中文](#中文) · [English](#english)

A progressive multi-agent software development workflow for Codex.

> **Use the lightest process that is sufficient for the task.** Small changes stay fast; full multi-agent orchestration is reserved for work that actually benefits from it.

---

# 中文

## 简介

`multi-agent-development` 是一个面向 **Codex** 的软件开发 Skill。

新版核心原则不是“所有代码修改都走完整多代理流程”，而是：

> **小事别折腾，大事再上流程；谁负责改代码，谁就自己先看代码。**

Skill 会从三档工作流中选择最轻且足够安全的一档：

```text
FAST
STANDARD
ORCHESTRATED
```

## 最重要的设计原则

### Exploration 必须存在，但 Explorer 不是必需的

AI 编码本身就需要搜索、阅读、理解代码。

因此普通任务不再这样：

```text
Explorer
  ↓
输出分析
  ↓
Developer 重新读代码
  ↓
实现
```

而是：

```text
Developer
  ↓
自己搜索 / 阅读 / 理解
  ↓
直接实现
```

只有当 Controller 需要知道**如何拆分多个任务、谁依赖谁、哪些文件会冲突、哪些任务能并行**时，才单独启动 Explorer。

一句话：

> **Exploration is mandatory; Explorer is optional.**

## FAST

适合局部、明确、低风险、容易验证的小修改。

例如：

- 改几个局部逻辑点；
- 增加一个简单 UI / 配置项；
- 修改少量字段；
- 一个明显且确定的简单 Bug；
- 几十行左右的普通局部修改。

流程：

```text
主 Codex
  ↓
搜索 / 阅读相关代码
  ↓
直接修改
  ↓
Targeted Build / Test
  ↓
检查最终 Diff
  ↓
完成
```

FAST 默认：

- 主会话可以直接修改生产代码；
- 不创建 Explorer；
- 不创建 Developer 子代理；
- 不强制 Reviewer；
- 不创建 Agent Pool；
- 不创建 worktree / staging；
- 不强制临时分支；
- 不执行完整 SHA 状态机。

如果实现过程中发现范围或风险比预期大，则升级到 STANDARD / ORCHESTRATED。

## STANDARD

适合比小改动复杂，但仍然应该由**一个 Developer 从头到尾负责**的任务。

流程：

```text
Controller
  ↓
简短 Task Brief
  ↓
Developer
  ├─ 自己探索代码
  ├─ 自己实现
  └─ 自己验证
  ↓
必要时 Independent Reviewer
  ↓
Controller Final Check
```

STANDARD 默认：

- 不单独启动 Explorer；
- Developer 自己完成探索 + 实现；
- 使用轻量 Task Brief；
- 不使用 Agent Pool；
- 不使用 staging；
- 不默认创建 worktree；
- 不强制完整 `TASK_HEAD / VALIDATED_HEAD / REVIEWED_HEAD / ACCEPTED_HEAD` 状态机；
- Reviewer 按风险决定是否启用，而不是一刀切。

适合 Review 的情况包括：中高风险、公共行为/API、错误处理、持久化、并发/生命周期、安全、测试覆盖较弱或 Developer 明确存在不确定性。

## ORCHESTRATED

只有真正需要治理和并行价值时才启用完整多代理流程。

典型情况：

- 多个 Developer 可以安全并行；
- 跨多个耦合模块；
- 需要 Dependency / Hot-file Map；
- 高风险或大影响面；
- 复杂并发 / 生命周期 / 数据 / 协议修改；
- Bug 根因未知、偶发、难复现；
- 需要严格 worktree、staging 和 Integration Review。

流程：

```text
Controller
  ↓
Explorer / Bug Investigator（仅必要时）
  ↓
Dependency / Ownership Plan
  ↓
Task A / B / C
  ↓
Agent Pool + Worktrees（有并行时）
  ↓
Developers
  ↓
Per-task Validation + Independent Review
  ↓
Staging
  ↓
Full Validation
  ↓
Integration Reviewer
  ↓
Promote to User Target
  ↓
Cleanup
```

只有这档默认启用：

- Agent Pool；
- 多 worktree / task branches；
- Git Resource Ledger；
- Commit-bound SHA 状态机；
- staging branch；
- per-task independent review；
- Integration Review；
- 完整 cleanup gate。

## Explorer 到底什么时候用？

Explorer 的价值不是“替 Developer 先看一遍代码”，而是帮助 Controller **安全拆任务**。

它应该回答：

```text
哪些子系统受影响？
谁负责哪块？
哪些文件是 Hot Files？
Task A 是否依赖 Task B？
哪些任务可以并行？
最终按什么顺序集成？
```

不需要替 Developer 写一份完整实现教程。

如果只需要一个 Developer，就通常不需要 Explorer。

## Bugfix

### 简单 Bug

```text
同一个编码上下文
→ 查代码
→ 找到原因
→ 修
→ 回归验证
```

### 普通 Bug

```text
一个 Developer
→ 自己调查
→ 自己实现
→ 验证
→ 必要时 Review
```

### 难 Bug

根因本身就是独立难题时，才使用 Bug Investigator：

```text
Bug Investigator
→ Evidence / Candidate Causes / Root Cause
→ Developer
→ Fix
→ Regression Verification
```

## 模型路由

模型路由只在**真的使用子代理时**生效。

- FAST：直接使用当前主会话模型，不为了模型路由额外创建 Agent；
- STANDARD：通常一个 Developer，按实际风险选择模型；Reviewer 仅必要时启动；
- ORCHESTRATED：使用完整 Role + Risk Adaptive Routing。

当前建议：

| 角色 | 默认/建议 |
| --- | --- |
| Explorer | Luna max |
| Bug Investigator | Luna max → Terra xhigh/max，必要时 Sol |
| Developer | 低风险 Luna max；普通生产代码 Terra xhigh；高风险 Terra max |
| Reviewer | Luna max；高风险/争议 Terra xhigh |
| Integration Reviewer | Terra xhigh；高风险 Terra max |

## 为什么这样更快？

因为新版避免了小任务中的重复上下文成本：

```text
旧：Controller 看一遍
    → Explorer 看一遍
    → Developer 再看一遍
    → Reviewer 再看一遍

新 FAST：主会话看一次并直接完成

新 STANDARD：Developer 看一次并直接完成
```

复杂治理机制仍然保留，但只在它的风险收益大于协调成本时启用。

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

Heavy references are intended to be loaded only when their phase/tier applies.

## 安装

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

如果设置了 `CODEX_HOME`：

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:CODEX_HOME\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

更新：

```bash
git pull
```

## 使用

```text
$multi-agent-development
帮我修改这个功能。优先选择最轻量的工作流，只有确实需要时才升级为多代理并行流程。
```

## License

MIT License. See [LICENSE](./LICENSE).

---

# English

## Overview

`multi-agent-development` is a progressive software-development Skill for Codex.

Its key rule is:

> **Use the lightest workflow that is sufficient for the task.**

It provides three tiers:

- **FAST** — main session explores, edits, validates, and inspects the diff directly;
- **STANDARD** — one Developer owns exploration + implementation + validation, with independent review only when justified;
- **ORCHESTRATED** — full multi-agent/worktree/staging/review workflow for complex, parallel, high-risk, or difficult diagnostic work.

## Exploration policy

Code exploration is a natural part of coding-agent work. A separate Explorer is not the default.

Use an Explorer only when the Controller needs a global map for decomposition, ownership, hot files, dependencies, parallelization, or integration order.

**Exploration is mandatory; Explorer is optional.**

## FAST

```text
main session
→ search/read
→ edit
→ targeted validation
→ inspect diff
→ done
```

No required subagents, Reviewer, Agent Pool, worktree, staging, temporary branch, or commit-state machine.

## STANDARD

```text
Controller brief
→ one Developer explores + implements + validates
→ optional Reviewer when risk justifies it
→ final check
```

No Explorer, Agent Pool, staging, or full Git certification by default.

## ORCHESTRATED

```text
optional Explorer / Bug Investigator
→ dependency/ownership plan
→ task Developers/worktrees
→ scoped validation + independent reviews
→ staging
→ final validation
→ Integration Reviewer
→ promotion
→ cleanup
```

This tier retains the repository's strict commit-bound and Git lifecycle protections.

## Model routing

Model routing applies only when delegated roles exist. FAST stays in the main coding context. STANDARD routes its single Developer by risk. ORCHESTRATED uses the full role+risk escalation policy.

## Installation

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

or:

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

## License

MIT License. See [LICENSE](./LICENSE).