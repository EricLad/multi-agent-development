# multi-agent-development

一个面向 **Codex** 的自适应软件开发 Skill。

`multi-agent-development` 会根据任务规模、风险、并行价值、执行歧义、任务持续长度和成本选择合适的开发流程。核心目标不是尽量多开 Agent，而是用**最少的编排成本**正确完成任务，并把强模型用在真正需要长程推理和跨系统理解的地方。

## 实现思路

Skill 将开发任务分为三个执行级别：

```text
FAST
STANDARD
ORCHESTRATED
```

同时采用 **Capability Routing（能力路由）**：

- 明确、边界清晰、成本敏感的实现任务优先使用 GPT-5.6 Terra；
- 长周期、跨系统、存在明显执行歧义或反复不收敛的任务优先考虑 GPT-6 Astra；
- 风险主要决定 Review / Validation / Git 隔离强度，而不是简单决定“模型越强越好”。

---

## FAST

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

Targeted checks 已经证明目标后，不会为了流程完整性继续重复全量测试；只有后续代码变化、失败或具体未解决风险使已有验证失效时才扩大验证。

---

## STANDARD

适合一个 Developer 可以独立完成，但修改范围和复杂度已经超过普通小改动的任务。

```text
Controller
  ↓
明确 Outcome + Boundary
  ↓
选择最合适的 Developer 模型
  ├─ Terra：明确、边界清晰、成本优先
  └─ Astra：长周期 / 跨系统 / 高执行歧义
  ↓
Developer 搜索代码 + 自主选择局部实现方式 + 实现 + 验证
  ↓
必要时 Reviewer
  ↓
完成
```

Developer 自己完成局部代码探索、实现和验证，不额外使用 Explorer。

Controller 不再需要提前决定每一个 helper、私有函数、文件组织或具体编辑顺序。只要下面这些真正影响结果的边界已经明确，Developer 就可以依据仓库现有代码自行决定可逆的局部实现细节：

```text
Goal
Scope
Architectural decisions / hard constraints（如有）
Non-goals
Validation
```

如果实现过程中发现关键架构假设不成立，只阻塞依赖这个决策的部分；不依赖该决策的安全工作可以继续进行。

适合：

- 一个完整但边界明确的功能；
- 一个子系统内的多文件修改；
- 普通重构；
- 一般复杂度的 Bug 修复；
- 一个 Agent 能长期保持完整上下文更有价值的任务。

---

## ORCHESTRATED

适合大型、跨模块、高风险、复杂 Bug 或真正适合并行开发的任务。

```text
Controller
  ↓
Explorer / Bug Investigator（按需）
  ↓
Dependency / Ownership / Critical Path Plan
  ↓
定义各 Task 的 Outcome + Boundary + Critical Invariants
  ↓
Agent Pool（仅在并行能缩短关键路径时）
  ↓
Capability-routed Developers
  ↓
Batch Review
  ↓
必要时同一 Developer Context 批量修复
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

Explorer 主要用于分析任务之间的代码所有权、依赖关系、Hot Files、关键路径、可并行范围和集成顺序。具体实现仍由各 Developer 在自己的任务范围内完成。

并行的判断标准不是“有空闲 Agent 就启动”，而是：

> 当两个或更多独立 Ready Task 位于关键路径、能够解锁关键路径或能显著缩短关键路径，并且预计节省的时间高于协调 / Review / Build / Merge 成本时，Controller 主动并行委派。

这也避免强模型因为自己能够处理长任务，就默认把所有事情串行完成。

ORCHESTRATED 会尽量一次性收集 Review 问题并批量修复，同时采用分层验证：开发阶段运行最相关的快速检查，Task 完成时进行任务级验证，昂贵的全量/集成/真实数据验证主要集中在 staging。

如果多轮 Review 仍持续产生新的重大问题，Controller 会先重新检查方案、任务边界、Critical Invariants 以及当前模型/推理强度是否合适，而不是无限循环修补。

---

## Plan Readiness

STANDARD 和 ORCHESTRATED 在启动 Developer 前，会尽量明确：

```text
Goal
Scope
Architectural decisions / hard constraints（仅真正重要的）
Non-goals
Validation
```

对于高风险 ORCHESTRATED Task，还可以补充少量 **Critical Invariants**，明确原子性、回滚、生命周期、兼容性等必须成立的关键规则。

### 不再要求 Controller 预编程

旧式做法容易把 `Implementation approach` 写得过细，导致 Controller 和 Developer 重复分析同一实现。

现在只要求 Controller 决定真正具有后果的事项，例如：

- 公共 API / 协议 / Schema；
- Ownership / Lifetime / Threading model；
- 安全边界；
- 持久化兼容性；
- 原子性 / 回滚语义；
- 跨任务共享接口。

Developer 自己决定：

- 使用哪个现有 helper；
- 局部函数如何组织；
- 哪些私有文件需要修改；
- exact implementation sequence；
- 其他可逆、低风险的局部实现细节。

---

## 模型路由

模型选择不再绑定成固定的“角色 → 单一模型”，而使用以下维度：

```text
EXECUTION_AMBIGUITY
TASK_HORIZON
INTEGRATION_BREADTH
COST_SENSITIVITY
RISK_LEVEL
```

其中 `RISK_LEVEL` 主要决定治理强度；前四项主要决定模型能力需求。

### 典型 Developer 路由

| 任务特征 | 建议 |
| --- | --- |
| 明确、边界清晰、Short/Medium horizon | **GPT-5.6 Terra medium** |
| Long horizon、Cross-system、明显执行歧义 | **GPT-6 Astra medium/high** |
| 并发 / 生命周期 / 状态机 / 数据一致性等困难语义 | Astra high/xhigh |
| 已有合理方案但连续多次不收敛 | 先检查边界 / Invariants，再考虑提高 Astra reasoning effort |
| Astra 不可用 | 按同一能力需求回退到合适的 GPT-5.6 Terra / Sol 路径 |

Terra 仍然是很重要的成本/性能执行模型；Astra 主要用于**认知成本高于编码成本**的任务，而不是替代所有 Terra Developer。

### 其他角色

- **Controller**：默认使用当前主会话；长周期、大范围 ORCHESTRATED 工作在可用且成本合理时更适合 Astra；
- **Explorer**：普通 locate/map 使用成本较低模型即可，困难跨模块架构推理再使用 Astra；
- **Bug Investigator**：普通问题使用较低成本模型，长周期多假设、并发/状态损坏问题倾向 Astra high/xhigh；
- **Reviewer**：按缺陷发现难度和风险选择，而不是看 Developer 用了什么模型；
- **Integration Reviewer**：普通集成可用 Terra 高推理档，复杂 Cross-system 集成倾向 Astra high/xhigh。

详细规则见 `references/model-routing.md`。

---

## Reasoning Effort 与上下文连续性

对于 Astra，如果运行时支持在保留上下文的情况下调整 reasoning effort，优先考虑：

```text
Astra medium
  ↓
high
  ↓
xhigh
  ↓
max（仅真正困难且已有证据支持时）
```

而不是：

```text
遇到困难
  ↓
丢弃现有上下文
  ↓
重新启动另一个 Agent
```

长期任务中，同一个 Developer 通常应继续负责兼容的 Review 修复，因为它已经拥有代码路径、失败假设和验证证据。

只有在以下场景才优先切换新 Context：

- 独立 Review；
- 并行所有权；
- Git 隔离；
- 不同角色；
- 需要刻意获得 fresh perspective；
- 原上下文已经不再可靠。

---

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

普通功能：

```text
$multi-agent-development
给用户管理模块增加批量禁用功能，并完成必要的测试。
```

通常会选择 **STANDARD**。如果任务已经明确且属于普通执行工作，优先走 Terra；如果发现任务其实需要长期保持大量上下文或跨系统推理，则可以切到 Astra。

复杂并行任务：

```text
$multi-agent-development
重构用户系统，包括数据库层、业务层和 UI。
请分析依赖关系，在安全的前提下并行开发。
```

如果适合拆分，会选择 **ORCHESTRATED**，沿关键路径组织真正有价值的并行 Developer，并通过批量 Review、分层验证和 Integration Review 完成集成。

复杂 Bug：

```text
$multi-agent-development
程序关闭用户会话时偶尔崩溃。
请先确认根因，再完成修复和回归验证。
```

如果根因未知、涉及并发或生命周期等复杂问题，会按需启用 Bug Investigator；长周期、多假设诊断在 Astra 可用时优先考虑 Astra。

也可以明确指定执行方式：

```text
$multi-agent-development
这是一个很小的修改，请优先使用 FAST。
```

或者：

```text
$multi-agent-development
这个任务涉及多个模块，请使用 ORCHESTRATED，并且只在能缩短关键路径时并行开发。
```

## License

MIT License. See [LICENSE](./LICENSE).
