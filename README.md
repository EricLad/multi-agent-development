# multi-agent-development

[中文](#中文) · [English](#english)

A multi-agent software development workflow for Codex.

> Make the main Codex session the **Controller / Tech Lead**, and delegate implementation to independent Explorer, Developer, Reviewer, and Integration Reviewer subagents.

---

# 中文

## 简介

`multi-agent-development` 是一个面向 **Codex** 的多代理软件开发 Skill。

它的核心思想是：**主会话不直接承担编码工作，而是作为 Controller / Tech Lead，专门负责需求确认、代码探索调度、任务拆分、任务分配、审查裁决、最终复核和代码合并；实际编码全部交给子代理完成。**

对于简单任务，Skill 会采用单 Developer + 独立 Reviewer 的流程；对于复杂任务，则先由 Explorer 摸清代码影响面，再将任务拆分到多个独立 worktree 中并行开发，最终在所有代码合并后增加一次 Integration Review 和完整验证。

## 为什么需要它

直接让一个 AI 会话从需求理解一路做到编码、测试和自我审查，容易出现几个问题：

- 开发者和审查者是同一个上下文，容易产生确认偏差；
- 在没有摸清代码结构之前就开始拆任务，容易错误并行；
- 多个代理可能同时修改同一组 hot files，导致冲突或语义不一致；
- 每个分支单独正确，不代表最终合并后的整体结果正确；
- 主会话长期参与底层编码后，容易被大量实现细节占用上下文。

这个 Skill 将开发过程拆成多个职责明确的角色，并通过 Build/Test、独立 Review、主控裁决和 Integration Review 建立质量门。

## 核心角色

| 角色 | 默认职责 | 优先模型配置 |
| --- | --- | --- |
| Controller / Tech Lead | 需求确认、复杂度判断、任务拆分、调度、裁决、合并、最终验收 | 当前主会话 |
| Explorer | 只探索代码，不修改生产代码；分析影响面、依赖、调用链和 hot files | 5.6 Terra xhigh |
| Developer | 在明确任务边界内实现代码并完成 scoped Build/Test | 5.6 Terra xhigh |
| Reviewer | 独立代码审查，不直接修改代码 | 5.6 Luna xhigh + Code Review |
| Integration Reviewer | 审查合并后的完整 `BASE_COMMIT..FINAL_HEAD` diff | 5.6 Luna xhigh + Code Review |

> 如果当前 Codex 环境无法选择完全相同的模型或模式，Skill 会要求使用最接近的可用配置，同时保留 Developer / Reviewer 的角色隔离和独立审查原则。

## 工作流

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

## 核心原则

1. **主会话原则上不修改生产代码。** 主会话专注于思考、调度、裁决和集成。
2. **复杂任务先探索，再拆分。** 不在不了解代码结构的情况下盲目并行。
3. **Developer != Reviewer。** 开发者不能作为自己改动的权威审查者。
4. **Reviewer 发现问题，Developer 修复或提供反证，Controller 最终裁决。**
5. **并行依据是代码所有权和依赖关系，而不是功能名称不同。**
6. **每个实现任务必须先通过 scoped Build/Test，才能进入 Review。**
7. **复杂任务合并后必须进行 Integration Review。** 单个 worktree 正确不代表整体集成正确。
8. **最终完成必须以验证证据为准，而不是以代理声明“完成”为准。**
9. **项目自身的 `AGENTS.md`、构建规则、测试规则和架构约束优先作为项目级规范。**
10. **只清理由本工作流创建的临时 worktree/branch，不删除用户已有工作或未提交修改。**

## Review 严重级别

Reviewer 使用统一的 Finding 严重度：

| 级别 | 含义 | 默认处理 |
| --- | --- | --- |
| BLOCKER | 编译失败、崩溃、数据损坏、安全漏洞等严重问题 | 必须修复 |
| HIGH | 高概率导致功能错误、稳定性问题或严重设计缺陷 | 必须修复 |
| MEDIUM | 边界情况或潜在问题 | Controller 裁决 |
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

进入 Skill 目录后拉取最新版本：

```bash
git pull
```

## 使用

可以显式调用：

```text
$multi-agent-development 帮我开发这个新功能：……
```

也可以直接描述开发需求。当任务符合 `SKILL.md` 中的触发条件时，Codex 可以自动选择这个 Skill。

### 示例

```text
$multi-agent-development
给用户管理模块增加批量禁用功能。先判断是否需要拆分；如果需要并行开发，请按照 Skill 的 worktree 和独立 review 流程执行。
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
    ├── explorer.md
    ├── integration-reviewer.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

其中：

- `SKILL.md`：Controller 的核心工作流和触发规则；
- `agents/openai.yaml`：Codex UI 元数据；
- `references/explorer.md`：代码探索代理规范；
- `references/task-contract.md`：开发任务契约；
- `references/worker.md`：Developer 执行规范；
- `references/reviewer.md`：独立代码审查规范；
- `references/integration-reviewer.md`：最终集成审查规范；
- `references/quality-and-git.md`：Build/Test、分支、worktree 和 Git 安全规则。

## 与项目 `AGENTS.md` 的关系

这个 Skill 负责定义**通用的软件开发工作流**；项目自己的 `AGENTS.md` 负责定义**项目级工程约束**。

例如：

```text
multi-agent-development
        ↓
通用多代理开发流程
        ↓
项目 AGENTS.md
        ↓
Qt / CMake / Rust / TypeScript / 测试 / 架构等项目规范
        ↓
Developer
```

两者不是替代关系，而是上下两层约束。

## 贡献

欢迎提交 Issue 或 Pull Request 来改进：

- Simple / Complex 的判断规则；
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

`multi-agent-development` is a multi-agent software development Skill for **Codex**.

Its central rule is simple: **the main Codex session acts as the Controller / Tech Lead and does not normally implement production code itself.** It owns requirement clarification, exploration orchestration, task decomposition, review arbitration, integration, and final acceptance, while coding work is delegated to specialized subagents.

For simple tasks, the workflow uses one Developer followed by an independent Reviewer. For complex work, an Explorer first maps the code impact and dependencies, implementation is split across isolated worktrees when safe, and the merged result is subjected to an independent Integration Review and full validation.

## Roles

| Role | Responsibility | Preferred configuration |
| --- | --- | --- |
| Controller / Tech Lead | Requirements, decomposition, orchestration, arbitration, merge, final acceptance | Main session |
| Explorer | Read-only code exploration, impact/dependency/hot-file analysis | 5.6 Terra xhigh |
| Developer | Implement a bounded task and run scoped validation | 5.6 Terra xhigh |
| Reviewer | Independent code review; does not directly edit the implementation | 5.6 Luna xhigh + Code Review |
| Integration Reviewer | Review the complete merged `BASE_COMMIT..FINAL_HEAD` change | 5.6 Luna xhigh + Code Review |

If an exact model or mode is unavailable, the Skill instructs Codex to use the closest capable configuration while preserving role separation and independent review.

## Workflow

### Simple

```text
requirements
  -> Developer
  -> scoped build/test
  -> independent Reviewer
  -> Developer fix or evidence-based dispute
  -> Controller arbitration/final review
  -> done
```

### Complex

```text
requirements
  -> Explorer
  -> impact/dependency analysis
  -> Controller decomposition
  -> parallel Developers in isolated worktrees
  -> scoped build/test
  -> independent per-task Reviewers
  -> Developer fix/dispute
  -> Controller arbitration
  -> merge
  -> Integration Reviewer
  -> clean/full validation
  -> Controller final review
  -> cleanup
```

## Principles

- The main session is a Controller, not the default implementation worker.
- Explore before decomposing complex work.
- Developer and Reviewer must be independent agents/contexts.
- Review findings require evidence and are arbitrated by the Controller when disputed.
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

Invoke it explicitly:

```text
$multi-agent-development Implement this feature: ...
```

Or describe a qualifying development task normally and allow Codex to select the Skill from its `SKILL.md` trigger description.

## Repository structure

```text
multi-agent-development/
├── README.md
├── LICENSE
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── explorer.md
    ├── integration-reviewer.md
    ├── quality-and-git.md
    ├── reviewer.md
    ├── task-contract.md
    └── worker.md
```

## Contributing

Issues and pull requests are welcome, especially for improvements to task classification, decomposition, worktree orchestration, review quality, integration review, validation strategies, and adaptation to new Codex agent/model capabilities.

Keep `SKILL.md` focused on the core workflow and place stage-specific detail in `references/` so the Skill continues to use Codex context efficiently.

## License

MIT License. See [LICENSE](./LICENSE).
