# multi-agent-development

一个面向 Codex 的多代理软件开发 Skill。

它把开发过程组织成一套固定的工程工作流：**主会话只负责需求确认、探索调度、任务拆分、审查裁决、最终复核和代码合并；实际编码全部交给子代理完成。**

## 核心流程

```text
需求
 ↓
主控确认
 ↓
复杂度判断
 ├─ Simple
 │   ↓
 │ Developer (Terra xhigh)
 │   ↓
 │ Build / Test
 │   ↓
 │ Independent Reviewer (Luna xhigh)
 │   ↓
 │ Developer 修复 / 主控裁决
 │   ↓
 │ 主控最终复核
 │
 └─ Complex
     ↓
   Explorer (Terra xhigh)
     ↓
   影响面 / 依赖 / hot files
     ↓
   主控拆分任务
     ↓
   多 Worktree 并行开发
     ↓
   各自 Build / Test
     ↓
   独立 Reviewer
     ↓
   Developer 修复 / 主控裁决
     ↓
   主控复核并合并
     ↓
   Integration Reviewer (Luna xhigh)
     ↓
   Clean Build / Full Test / Regression Check
     ↓
   主控最终复核
     ↓
   清理 Worktree
```

## 设计原则

- 主会话是 Controller / Tech Lead，原则上不直接修改实现代码。
- 复杂任务先探索，再拆分；不要在不了解代码结构时盲目并行。
- Developer 与 Reviewer 必须使用独立子代理和独立上下文。
- Reviewer 负责发现问题，Developer 负责修复或提供反证，主会话负责最终裁决。
- 并行依据是代码所有权和依赖关系，而不是功能名称看起来不同。
- 每个开发任务都必须经过 Build/Test 才能进入下一阶段。
- 复杂任务合并后必须进行 Integration Review，检查最终整体 diff。
- 项目自身的 `AGENTS.md` / 构建文档 / 测试规范始终优先作为项目级约束。

## 安装

仓库根目录本身就是 Skill 目录。

### Windows PowerShell

```powershell
git clone https://github.com/EricLad/multi-agent-development.git "$env:USERPROFILE\.codex\skills\multi-agent-development"
```

### Linux / macOS

```bash
git clone https://github.com/EricLad/multi-agent-development.git ~/.codex/skills/multi-agent-development
```

安装或更新后重新启动 Codex，使其重新发现 Skill。

## 使用

可以显式调用：

```text
$multi-agent-development 帮我开发这个新功能：……
```

安装后，当任务明显属于功能开发、较大范围代码修改、多代理并行开发、worktree 协作或独立代码审查时，Codex 也可以根据 `SKILL.md` 中的描述自动选择该 Skill。

## 模型路由

本 Skill 按当前工作流优先采用：

- Explorer：**5.6 Terra xhigh**
- Developer：**5.6 Terra xhigh**
- Reviewer：**5.6 Luna xhigh**，Code Review 模式
- Integration Reviewer：**5.6 Luna xhigh**，Code Review 模式

如果当前 Codex 环境无法选择这些精确模型或模式，主控应选择最接近的可用配置，并明确说明一次降级情况，而不是悄悄改变工作流角色边界。

## 目录

```text
multi-agent-development/
├── SKILL.md
├── README.md
├── agents/
│   └── openai.yaml
└── references/
    ├── workflow.md
    ├── explorer.md
    ├── task-contract.md
    ├── worker.md
    ├── reviewer.md
    ├── integration-reviewer.md
    └── quality-and-git.md
```

## 状态

当前版本以 Codex 的多代理开发和 worktree 工作流为主要目标，后续可以继续根据真实项目中的执行效果调整任务拆分策略、审查标准和模型路由。
