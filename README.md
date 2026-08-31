# multi-agent-development

一个面向 **Codex** 的自适应软件开发 Skill。

`multi-agent-development` 会根据任务规模、风险和并行价值选择合适的开发流程，并在委派开发前尽量把实现方案明确化，让子代理专注于高质量执行。

## 实现思路

Skill 将开发任务分为三个执行级别：

```text
FAST
STANDARD
ORCHESTRATED
```

### FAST

适合局部、明确、低风险、容易验证的小修改。

```text
主 Codex
  ↓
搜索 / 阅读相关代码
  ↓
直接修改
  ↓
Targeted Build / Test
  ↓
检查 Diff
  ↓
完成
```

FAST 模式由当前主会话直接完成代码探索、实现和验证，不额外创建 Developer、Reviewer、worktree 或 staging。

适合：

- 小范围功能修改；
- UI / 配置项调整；
- 少量字段或逻辑修改；
- 明确且容易验证的 Bug；
- 其他局部低风险改动。

---

### STANDARD

适合一个 Developer 可以独立完成，但修改范围和复杂度已经超过普通小改动的任务。

```text
Controller
  ↓
明确 Goal / Scope / Implementation / Non-goals / Validation
  ↓
GPT-5.6 Terra medium Developer
  ├─ 搜索 / 阅读局部代码
  ├─ 按方案实现
  └─ Build / Test
  ↓
必要时 Reviewer
  ↓
完成
```

Developer 自己完成局部代码探索、实现和验证，不额外使用 Explorer。

在启动 Developer 前，Controller 会尽量确保方案已经具备可执行性；如果实现过程中发现关键假设不成立，Developer 返回 Controller 重新规划，而不是自行扩大设计和修改范围。

适合：

- 一个完整但边界明确的功能；
- 一个子系统内的多文件修改；
- 普通重构；
- 一般复杂度的 Bug 修复。

---

### ORCHESTRATED

适合大型、跨模块、高风险、复杂 Bug 或适合并行开发的任务。

```text
Controller
  ↓
Explorer / Bug Investigator（按需）
  ↓
Dependency / Ownership Plan
  ↓
把 Task A / B / C 的方案分别明确
  ↓
Agent Pool
  ↓
Terra medium Developer A / B / C
  ↓
Independent Review
  ↓
Staging
  ↓
Full Validation
  ↓
Integration Review
  ↓
合并到目标分支
  ↓
Cleanup
```

该模式会根据实际需要启用：

- Explorer / Bug Investigator；
- 多 Developer 并行开发；
- Agent Pool 并发调度；
- Git worktree；
- 独立 Reviewer；
- Commit-bound validation；
- staging branch；
- Integration Review；
- 自动清理临时 worktree / branch。

Explorer 主要用于分析任务之间的代码所有权、依赖关系、Hot Files、可并行范围和集成顺序。具体实现仍由各 Developer 在自己的任务范围内完成。

---

## Plan Readiness

STANDARD 和 ORCHESTRATED 在启动 Developer 前，会尽量明确以下内容：

```text
Goal
Scope
Implementation approach
Non-goals
Validation
```

方案明确后，默认使用 **GPT-5.6 Terra medium** 进行开发。

如果方案与实际代码存在关键冲突，优先返回 Controller 重新规划；只有实现本身仍存在明显复杂歧义时，才升级 Developer 模型。

## 模型路由

| 角色 | 默认策略 |
| --- | --- |
| Explorer | Luna max |
| Bug Investigator | Luna max → Terra xhigh / max |
| Developer | **GPT-5.6 Terra medium**；存在明显执行歧义时再升级 Terra xhigh / max |
| Reviewer | Luna max；高风险或存在重大争议时 Terra xhigh |
| Integration Reviewer | Terra xhigh；高风险集成 Terra max |
| Critical Escalation | Sol xhigh / max |

任务风险主要决定验证和 Review 强度；Developer 是否升级主要取决于实现方案是否仍存在难以解决的歧义。

FAST 模式不会为了模型路由额外创建子代理，直接使用当前主会话模型。

## 安装

Codex 用户级 Skill 默认目录为：

```text
~/.codex/skills/
```

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

如果设置了 `CODEX_HOME`：

```bash
git clone https://github.com/EricLad/multi-agent-development.git "$CODEX_HOME/skills/multi-agent-development"
```

安装完成后，如果 Codex 没有立即发现 Skill，请重新启动 Codex。

### 更新

进入 Skill 目录执行：

```bash
git pull
```

## 使用示例

小功能：

```text
$multi-agent-development
帮我给设置页面增加一个“自动检查更新”的选项。
```

通常会选择 **FAST**。

---

普通功能：

```text
$multi-agent-development
给用户管理模块增加批量禁用功能，并完成必要的测试。
```

通常会选择 **STANDARD**，由 Controller 明确方案后交给 Terra medium Developer 完成。

---

复杂并行任务：

```text
$multi-agent-development
重构用户系统，包括数据库层、业务层和 UI。
请分析依赖关系，在安全的前提下并行开发。
```

如果适合拆分，会选择 **ORCHESTRATED**，先明确各子任务方案，再并行交给多个 Developer 实现。

---

复杂 Bug：

```text
$multi-agent-development
程序关闭用户会话时偶尔崩溃。
请先确认根因，再完成修复和回归验证。
```

如果根因未知、涉及并发或生命周期等复杂问题，会按需启用 **Bug Investigator**。

---

也可以明确指定执行方式：

```text
$multi-agent-development
这是一个很小的修改，请优先使用 FAST。
```

或者：

```text
$multi-agent-development
这个任务涉及多个模块，请使用 ORCHESTRATED 并行开发。
```

## License

MIT License. See [LICENSE](./LICENSE).
