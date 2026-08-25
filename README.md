# multi-agent-development

[中文](#中文) · [English](#english)

A multi-agent software development workflow for Codex.

> Make the main Codex session the **Controller / Tech Lead**, and delegate exploration, implementation, bug investigation, independent review, and integration review to specialized subagents.

---

# 中文

## 简介

`multi-agent-development` 是一个面向 **Codex** 的多代理软件开发 Skill。

它的核心思想是：**主会话不直接承担编码工作，而是作为 Controller / Tech Lead，专门负责需求确认、复杂度判断、代码探索调度、任务拆分、任务分配、审查裁决、最终复核和代码合并；实际编码全部交给子代理完成。**

这个 Skill 不只用于开发新功能，也适用于：

- Feature 开发；
- Bug 修复；
- Refactor；
- Performance 优化；
- Maintenance 和其他非平凡代码修改。

## 核心角色

| 角色 | 默认职责 | 优先模型配置 |
| --- | --- | --- |
| Controller / Tech Lead | 需求确认、复杂度判断、任务拆分、调度、裁决、合并、最终验收 | 当前主会话 |
| Explorer / Bug Investigator | 只探索代码，不修改生产代码；分析影响面、依赖、调用链、hot files，或定位 Bug 根因 | 5.6 Terra xhigh |
| Developer | 在明确任务边界内实现代码并完成 scoped Build/Test | 5.6 Terra xhigh |
| Reviewer | 独立代码审查，不直接修改代码 | 5.6 Luna xhigh + Code Review |
| Integration Reviewer | 审查合并后的完整 `BASE_COMMIT..FINAL_HEAD` diff | 5.6 Luna xhigh + Code Review |

> 如果当前 Codex 环境无法选择完全相同的模型或模式，Skill 会要求使用最接近的可用配置，同时保留 Developer / Reviewer 的角色隔离和独立审查原则。

## 开发工作流

### Simple Task

适合修改范围明确、耦合低、不需要并行拆分的任务：

```text
Requirement
    ↓
Controller
    ↓
Developer
    ↓
Scoped Build / Test
    ↓
Independent Reviewer
    ↓
Developer Fix / Evidence-based Dispute
    ↓
Controller Arbitration & Final Review
    ↓
Done
```

简单任务通常不需要额外创建 worktree，但建议保留清晰的 branch 或 commit 边界，方便 diff、review 和回退。

### Complex Task

适合跨模块、有共享状态或公共接口、存在明显回归风险，或者能够安全并行开发的任务：

```text
Requirement
    ↓
Controller
    ↓
Explorer
    ↓
Impact / Dependency / Hot-file Analysis
    ↓
Controller Task Decomposition
    ↓
┌──────────────┬──────────────┬──────────────┐
│ Worktree A   │ Worktree B   │ Worktree C   │
│ Developer A  │ Developer B  │ Developer C  │
└──────┬───────┴──────┬───────┴──────┬───────┘
       ↓              ↓              ↓
    Build/Test      Build/Test      Build/Test
       ↓              ↓              ↓
   Reviewer A      Reviewer B      Reviewer C
       ↓              ↓              ↓
 Developer Fix    Developer Fix    Developer Fix
       └──────────────┬──────────────┘
                      ↓
               Controller Review
                      ↓
                    Merge
                      ↓
             Integration Reviewer
                      ↓
          Clean Build / Full Test
                      ↓
             Regression Check
                      ↓
            Controller Final Gate
                      ↓
                 Worktree Cleanup
```

## Bugfix Workflow

Bug 修复继续使用同一个 Skill，但 **Bug 的复杂度不能只看最终改了多少行代码**。

一个只需要改一行代码的 Bug，如果根因未知、偶发、涉及并发/生命周期、跨模块或难以验证，仍然属于 Complex。

### Simple Bugfix

适用于根因明确、范围局部、验证直接的 Bug：

```text
Bug Symptom
    ↓
Controller
    ↓
Developer
    ↓
Root Cause + Fix
    ↓
Regression Verification
    ↓
Build / Test
    ↓
Independent Reviewer
    ↓
Controller Final Review
    ↓
Confirmed Fix
```

### Complex Bugfix

适用于根因未知、偶发、跨模块、高风险或难以验证的 Bug：

```text
Bug Symptom / Evidence
    ↓
Controller
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
Controller Task Contract
    ↓
Developer Fix
    ↓
Regression Verification
    ↓
Independent Reviewer
    ↓
Integration Review when applicable
    ↓
Controller Final Gate
```

Bugfix 的核心原则是：

> **先区分现象和根因，再修复根因，最后验证原始故障不会回归。**

Developer 的 Bugfix 交付必须说明：

```text
Symptom
Root Cause
Evidence
Fix
Regression Verification
Residual Risk
```

并且精确声明结果属于：

- `Confirmed fix`：根因有证据支持，回归验证通过；
- `Mitigation`：降低了影响，但没有完全消除或证明根因；
- `Diagnostic change`：只增加了诊断能力，Bug 尚未修复；
- `Hypothesis-driven fix`：修复基于合理假设，但因果证据仍不完整。

Skill 不允许把“症状暂时消失”直接当成“Bug 已确认修复”。

## 核心原则

1. **主会话原则上不修改生产代码。** 主会话专注于思考、调度、裁决和集成。
2. **复杂任务先探索，再拆分。** 不在不了解代码结构的情况下盲目并行。
3. **Developer != Reviewer。** 开发者不能作为自己改动的权威审查者。
4. **Reviewer 发现问题，Developer 修复或提供反证，Controller 最终裁决。**
5. **并行依据是代码所有权和依赖关系，而不是功能名称不同。**
6. **每个实现任务必须先通过 scoped Build/Test，才能进入 Review。**
7. **复杂任务合并后必须进行 Integration Review。** 单个 worktree 正确不代表整体集成正确。
8. **Bugfix 必须尽可能证明 Root Cause，并进行 Regression Verification。**
9. **最终完成必须以验证证据为准，而不是以代理声明“完成”为准。**
10. **项目自身的 `AGENTS.md`、构建规则、测试规则和架构约束优先作为项目级规范。**
11. **只清理由本工作流创建的临时 worktree/branch，不删除用户已有工作或未提交修改。**

## Review 严重级别

| 级别 | 含义 | 默认处理 |
| --- | --- | --- |
| BLOCKER | 编译失败、崩溃、数据损坏、安全漏洞等严重问题 | 必须修复 |
| HIGH | 高概率导致功能错误、稳定性问题或严重设计缺陷 | 必须修复 |
| MEDIUM | 边界情况、回归验证不足或潜在问题 | Controller 裁决 |
| LOW | 轻微问题 | 通常不阻塞 |
| NIT | 命名、风格等建议 | 不阻塞 |

Developer 可以对 Finding 给出：

- `Confirmed`：确认问题并修复；
- `Disputed + Evidence`：不同意结论，并提供代码、测试、调用链或其他证据。

Developer 无权自行宣布 Reviewer 的重要 Finding 无效；存在争议时由 Controller 最终裁决。

## 安装

Codex 的用户级 Skill 默认位于 `$CODEX_HOME/skills`；如果没有设置 `CODEX_HOME`，默认目录是 `~/.codex/skills`。

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

如果你设置了 `CODEX_HOME`：

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:CODEX_HOME\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

如果你设置了 `CODEX_HOME`：

```bash
git clone https://github.com/EricLad/multi-agent-development.git "$CODEX_HOME/skills/multi-agent-development"
```

安装后如果 Codex 没有立即发现 Skill，请重新启动 Codex。

## 更新

进入 Skill 目录后：

```bash
git pull
```

## 使用

开发新功能：

```text
$multi-agent-development
给用户管理模块增加批量禁用功能。先判断是否需要拆分；如果需要并行开发，请按照 Skill 的 worktree 和独立 review 流程执行。
```

修复 Bug：

```text
$multi-agent-development
程序偶尔在关闭用户会话时崩溃。请先判断 Bug 的复杂度；如果根因不明确，先进行只读调查并证明 Root Cause，再交给 Developer 修复，并完成 Regression Verification 和独立 Review。
```

也可以直接描述符合条件的开发任务，让 Codex 根据 `SKILL.md` 的触发描述自动选择该 Skill。

## 项目结构

```text
multi-agent-development/
├── README.md
├── LICENSE
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── bugfix.md
    ├── explorer.md
    ├── integration-reviewer.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

其中：

- `SKILL.md`：Controller 的核心路由、工作流和触发规则；
- `agents/openai.yaml`：Codex UI 元数据；
- `references/bugfix.md`：Bug 调查、Root Cause Gate 和 Regression Verification 规范；
- `references/explorer.md`：代码探索代理规范；
- `references/task-contract.md`：开发任务契约；
- `references/worker.md`：Developer 执行规范；
- `references/reviewer.md`：独立代码审查规范；
- `references/integration-reviewer.md`：最终集成审查规范；
- `references/quality-and-git.md`：Build/Test、分支、worktree 和 Git 安全规则。

## 与项目 `AGENTS.md` 的关系

这个 Skill 负责定义**通用的软件开发工作流**；项目自己的 `AGENTS.md` 负责定义**项目级工程约束**。

```text
multi-agent-development
        ↓
通用多代理开发 / Bugfix 流程
        ↓
项目 AGENTS.md
        ↓
项目语言 / 构建 / 测试 / 架构规范
        ↓
Developer
```

## 贡献

欢迎提交 Issue 或 Pull Request 来改进：

- Simple / Complex 判断规则；
- Bug Root Cause 分析；
- Regression Verification；
- 多代理任务拆分策略；
- worktree 并行开发方式；
- Review Finding 规则；
- Integration Review；
- 不同语言和技术栈的验证策略；
- Codex 新模型和新代理能力的适配。

修改工作流时，请尽量保持 `SKILL.md` 精简，把角色细节和阶段性规则放入 `references/`，避免不必要地占用 Codex 上下文。

## License

MIT License. See [LICENSE](./LICENSE).

---

# English

## Overview

`multi-agent-development` is a multi-agent software development Skill for **Codex**. It supports feature development, bug fixing, refactoring, performance work, and maintenance.

The main Codex session acts as the **Controller / Tech Lead**. It owns clarification, classification, decomposition, orchestration, review arbitration, integration, and final acceptance, while implementation is delegated to specialized subagents.

## Preferred roles

| Role | Responsibility | Preferred configuration |
| --- | --- | --- |
| Controller / Tech Lead | Requirements, classification, orchestration, arbitration, merge, final acceptance | Main session |
| Explorer / Bug Investigator | Read-only code exploration, impact analysis, or root-cause investigation | 5.6 Terra xhigh |
| Developer | Implement a bounded task and run scoped validation | 5.6 Terra xhigh |
| Reviewer | Independent code review | 5.6 Luna xhigh + Code Review |
| Integration Reviewer | Review the complete merged `BASE_COMMIT..FINAL_HEAD` change | 5.6 Luna xhigh + Code Review |

## Bugfix workflow

Bug complexity is based on **diagnosis and validation difficulty**, not diff size.

Simple bugfix:

```text
symptom -> Developer -> root cause/fix -> regression verification -> build/test -> independent Reviewer -> Controller final gate
```

Complex bugfix:

```text
symptom/evidence
  -> Bug Investigator
  -> reproduction/evidence
  -> candidate root causes
  -> discriminating checks
  -> root cause conclusion
  -> Developer
  -> regression verification
  -> independent Reviewer
  -> Integration Review when applicable
  -> Controller final gate
```

A Bugfix handoff should state:

- Symptom
- Root Cause
- Evidence
- Fix
- Regression Verification
- Residual Risk

Completion states are intentionally precise: **Confirmed fix**, **Mitigation**, **Diagnostic change**, or **Hypothesis-driven fix**.

## Principles

- The main session is a Controller, not the default implementation worker.
- Explore before decomposing complex work.
- Developer and Reviewer must be independent agents/contexts.
- For bugs, distinguish symptom from root cause and require regression verification whenever practical.
- Parallelism is based on safe code ownership and dependency boundaries.
- Every implementation task must pass scoped build/tests before review.
- Complex merged changes require an Integration Review over the final combined diff.
- Completion is based on validation evidence, not an agent simply saying “done”.
- Repository-level `AGENTS.md`, build instructions, test rules, and architecture constraints remain authoritative project-specific guidance.

## Installation

User-level Codex skills live under `$CODEX_HOME/skills`. If `CODEX_HOME` is not set, Codex defaults to `~/.codex/skills`.

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

Restart Codex if the newly installed Skill is not discovered immediately.

## Usage

Feature:

```text
$multi-agent-development Implement this feature: ...
```

Bugfix:

```text
$multi-agent-development Investigate and fix this bug. Prove the root cause when possible, add regression verification, and use an independent Reviewer.
```

## Repository structure

```text
multi-agent-development/
├── README.md
├── LICENSE
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── bugfix.md
    ├── explorer.md
    ├── integration-reviewer.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

## Contributing

Issues and pull requests are welcome, especially for improvements to task classification, root-cause analysis, regression verification, decomposition, worktree orchestration, review quality, integration review, validation strategies, and adaptation to new Codex capabilities.

Keep `SKILL.md` focused on the core workflow and place stage-specific detail in `references/` so the Skill continues to use Codex context efficiently.

## License

MIT License. See [LICENSE](./LICENSE).
