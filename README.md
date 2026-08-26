# multi-agent-development

一个面向 **Codex** 的自适应软件开发 Skill。

`multi-agent-development` 会根据任务规模、风险和并行价值，自动选择合适的开发流程，在简单任务的开发效率与复杂任务的工程可靠性之间取得平衡。

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
Developer
  ├─ 搜索 / 阅读代码
  ├─ 实现
  └─ Build / Test
  ↓
必要时 Reviewer
  ↓
完成
```

Developer 自己完成代码探索、实现和验证，不额外使用 Explorer。

根据任务风险和不确定性，Controller 可以增加独立 Reviewer。

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
Task A / Task B / Task C
  ↓
Agent Pool
  ↓
Developer A / B / C
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

Explorer 主要用于分析任务之间的：

- 代码所有权；
- 依赖关系；
- Hot Files；
- 可并行范围；
- 集成顺序。

具体代码实现仍由各 Developer 在自己的任务范围内完成。

---

## 自适应模型路由

使用子代理时，Skill 会根据角色和任务风险选择模型，并在必要时升级。

| 角色 | 默认策略 |
| --- | --- |
| Explorer | Luna max |
| Bug Investigator | Luna max → Terra xhigh / max |
| Developer | 低风险 Luna max；普通任务 Terra xhigh；高风险 Terra max |
| Reviewer | Luna max；高风险或存在重大争议时 Terra xhigh |
| Integration Reviewer | Terra xhigh；高风险集成 Terra max |
| Critical Escalation | Sol xhigh / max |

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

在 Codex 中直接调用：

```text
$multi-agent-development
帮我给设置页面增加一个“自动检查更新”的选项。
```

对于这种局部低风险任务，通常会自动选择 **FAST**。

---

普通功能：

```text
$multi-agent-development
给用户管理模块增加批量禁用功能，并完成必要的测试。
```

如果一个 Developer 可以独立完成，通常会选择 **STANDARD**。

---

复杂并行任务：

```text
$multi-agent-development
重构用户系统，包括数据库层、业务层和 UI。
请分析依赖关系，在安全的前提下并行开发。
```

如果任务适合拆分，会选择 **ORCHESTRATED**，并根据需要使用 Explorer、多个 Developer、worktree、Reviewer 和 Integration Review。

---

复杂 Bug：

```text
$multi-agent-development
程序关闭用户会话时偶尔崩溃。
请先确认根因，再完成修复和回归验证。
```

如果根因未知、涉及并发或生命周期等复杂问题，会根据需要启用 **Bug Investigator**。

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
