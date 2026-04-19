# Terminology And Ownership

这份文档按模块整理关键术语，并说明它们属于谁、不属于谁。

它不替代各模块 `README.md`，而是作为跨模块归属索引使用。

## Harness Terms

`Harness` 负责 interaction 推进、runtime 控制和模型适配。

| Term | Meaning | Owned By | Not Owned By | Notes |
| --- | --- | --- | --- | --- |
| `AgentRuntime` | 从输入到终态的 turn/interaction 执行内核 | `Harness` | `Session`, `Gateway` | runtime 是 harness 的一部分，不是独立顶层模块 |
| `HarnessInstance` | 某个 session-bound harness worker 的实例身份对象 | `Harness` | `Session`, `Gateway` | gateway 管理实例；runtime 是实例内部 loop |
| `Gateway` | channel 与 harness 之间的双向交互边界 | `Harness` | `Session`, `Orchestration` | 负责 input normalization、control routing、egress projection |
| `ChannelAdapter` | 具体 channel 的协议边缘适配层 | `Harness` | `Session`, `Tools` | 在 gateway 外缘，不负责 turn eval |
| `InteractionLoop` | gateway 与 runtime 之间的标准交互循环 | `Harness` | `Session`, `Sandbox` | supplement input 属于当前 interaction |
| `ModelProviderAdapter` | 吸收模型厂商/API 差异的统一适配层 | `Harness` | `Tools`, `Session` | 是模型差异的唯一主吸收层 |
| `ContextGovernance` | context budget、compact、overflow recovery 的治理层 | `Harness` | `Session` | 不等于 transcript 或 memory store |
| `TaskManager` | 本地 task-driven runtime 的任务生命周期与恢复接口 | `Harness` | `Session`, `Orchestration` | task 不是 transcript |
| `BackgroundAgent` | 长于单 turn 生命周期的后台 agent | `Harness` | `Orchestration` | local 默认由 harness 管理生命周期 |
| `VerifierTask` | 评估/校验型运行时单元 | `Harness` | `Tools` | capability surface 仍可由 tools 暴露 |
| `WorkAllocator` | 本地多 agent / 多 worker 的分工与阻塞关系接口 | `Harness` | `Session` | 不等于 runtime task lifecycle |

## Session Terms

`Session` 负责 durable state、restore 和 session 内记忆链接。

| Term | Meaning | Owned By | Not Owned By | Notes |
| --- | --- | --- | --- | --- |
| `Transcript` | append-only 会话历史 | `Session` | `Harness`, `Tools` | transcript 不是 memory |
| `SessionCheckpoint` | append/read/resume 之间的标准衔接对象 | `Session` | `Gateway`, `Harness` | 对外暴露 durable 进度位置 |
| `ResumeSnapshot` | resume 时恢复 working state 的结构化快照 | `Session` | `Harness` | 供 runtime 重建当前可继续状态 |
| `ShortTermSessionMemory` | 当前 session continuity object | `Session` | `Harness`, `Orchestration` | 用于 compact / resume / away summary |
| `Durable Memory` | 可跨 turn / 跨 session recall 的长期沉淀 | `Session` via `session.memory` | `Harness` | 不承担 restore 责任 |
| `Scoped Durable Memory` | 有 user/project/agent/local 等作用域的 durable memory | `Session` via `session.memory` | `Gateway` | 是 durable memory 的作用域层 |

## Tools Terms

`Tools` 负责 capability declaration、execution 和 policy。

| Term | Meaning | Owned By | Not Owned By | Notes |
| --- | --- | --- | --- | --- |
| `ToolDefinition` | 模型可见的工具声明 | `Tools` | `Sandbox`, `Session` | 包含 schema、description、permission hooks |
| `ToolRegistry` | 可见工具集合与解析层 | `Tools` | `Harness` | harness 消费 registry，不拥有它 |
| `ToolExecutor` | 批量工具执行编排 | `Tools` | `Gateway`, `Orchestration` | 不等于 sandbox |
| `StreamingToolExecutor` | 增量 tool execution 与 progress/result 回流层 | `Tools` | `Gateway` | progress 在这里产生 |
| `ToolProgress` | tool 执行中的 progress update | `Tools` emits, `Gateway` projects | `Session` | 发射归 tools，外投归 gateway |
| `Command` | `Tools` 域内共享对象模型 | `Tools` | top-level module | 不是第六个顶层模块 |
| `Skill` | 提示词/工作流型能力 | `Tools` | `Harness` | 可由 tool bridge 暴露给模型 |
| `MCP` | 外部协议接入能力 | `Tools` | `Gateway` | client / adapter 属于 tools 域 |
| `VerificationCommand` | 对 harness 暴露的 command-like review capability | `Tools` | `Orchestration` | capability surface 在 tools，执行后端可委托 harness 或 orchestration |

## Sandbox Terms

`Sandbox` 负责执行隔离和安全边界。

| Term | Meaning | Owned By | Not Owned By | Notes |
| --- | --- | --- | --- | --- |
| `ExecutionSandbox` | 命令或代码执行隔离层 | `Sandbox` | `Tools` | tool 可以使用 sandbox，但不拥有它 |
| `EnvironmentSandbox` | 环境能力与资源边界层 | `Sandbox` | `Session` | 负责网络、凭证、文件系统边界 |
| `SecurityBoundary` | 可执行能力的结构化安全描述 | `Sandbox` | `Gateway` | 供 UI / agent / policy 使用 |
| `CapabilityModel` | sandbox 能做什么、不能做什么的统一模型 | `Sandbox` | `Harness` | 是执行面能力，不是 tool capability |

## Orchestration Terms

`Orchestration` 负责 cloud 托管控制面。

| Term | Meaning | Owned By | Not Owned By | Notes |
| --- | --- | --- | --- | --- |
| `ManagedAgentControlPlane` | cloud 托管 agent 的控制面 | `Orchestration` | `Harness` | 负责 wake、resume、provision、route |
| `WakeCoordinator` | 托管执行恢复与唤醒协调器 | `Orchestration` | `Session` | 不等于 turn runtime |
| `ProvisioningPlanner` | execution target 的按需创建与替换策略 | `Orchestration` | `Sandbox` | 只管 provision 计划，不等于 sandbox 本体 |
| `HostingProfile` | Local/Cloud 的职责分布模式 | shared deployment mapping | `Harness` | 影响部署位置，不改变五大模块边界 |
| `Host Scheduler` | 周期性 tick / due-job 推进器 | host or `Orchestration` | `Gateway` | 不是 gateway 核心职责 |

## Shared / Cross-Cutting Terms

这些概念跨模块使用，但仍有主归属。

| Term | Meaning | Owned By | Not Owned By | Notes |
| --- | --- | --- | --- | --- |
| `RuntimeEvent` | 执行过程中的标准事件对象 | shared object model | n/a | 被 harness、tools、gateway、session 共用 |
| `TerminalState` | 一次 interaction/turn 的终态描述 | shared object model | n/a | 由 runtime 产出，供外层消费 |
| `CapabilitySurface` | 宿主/模型可见能力投影面 | shared capability layer | top-level module | 不是五大核心模块之一 |

## Common Confusions

- `Gateway` vs `ChannelAdapter`
  `ChannelAdapter` 处理 channel-native 协议；`Gateway` 处理统一 input/control/projection 语义。

- `Gateway` vs `AgentRuntime`
  `Gateway` 协调入口和外投；`AgentRuntime` 推进 turn 本体。

- `HarnessInstance` vs `AgentRuntime`
  前者是 gateway 可管理的 worker identity；后者是该 worker 内部的 turn loop。

- `SessionHandle` vs `HarnessInstance`
  前者是 adapter/gateway 侧承载句柄；后者是运行 worker 的规范身份对象。

- `Transcript` vs `ShortTermSessionMemory`
  transcript 是原始 durable history；short-term memory 是 continuity summary。

- `ShortTermSessionMemory` vs `Durable Memory`
  前者服务当前 session continuity；后者服务跨 turn / 跨 session recall。

- `ToolProgress` emission vs projection
  `ToolProgress` 由 `Tools` 产生；由 `Gateway` 投影给 channel/UI。

- `VerificationCommand` vs `VerifierTask`
  前者是暴露给 harness/agent 的 capability surface；后者是其默认执行后端中的本地运行时对象。

- `Orchestration` vs `Host Scheduler`
  orchestration 定义任务与控制面语义；host scheduler 只是推进 due jobs 的宿主执行机制。
