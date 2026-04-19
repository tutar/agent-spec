# Module Overview

这份文档只关注五个核心模块的大边界。

它的目标不是替代各模块 `README.md`，而是帮助读者先回答三个问题：

- 这个模块负责什么
- 这个模块有哪些核心对象
- 这个模块不负责什么

## Harness

### 职责

`Harness` 负责把一次外部输入推进成完整的 agent interaction。

它负责：

- turn / interaction 推进
- model input 组装
- model provider 适配
- tool continuation 与 runtime event 发射
- `Gateway`、`AgentRuntime`、`ModelProviderAdapter` 等执行面子层

### 核心对象

- `AgentRuntime`
- `Gateway`
- `ChannelAdapter`
- `InteractionLoop`
- `ModelProviderAdapter`
- `ContextGovernance`
- `TaskManager`
- `BackgroundAgent`
- `VerifierTask`
- `WorkAllocator`

### 不属于该模块

- durable transcript storage
- session restore store
- tool registry / policy engine 本体
- execution sandbox
- cloud control plane

## Session

### 职责

`Session` 是 agent runtime 的 durable state boundary。

它负责：

- transcript / event log
- working state restore
- checkpoint / cursor / resume
- session 内短期连续性记忆
- `session.memory` 下的 durable memory linkage 与 recall/consolidation 接口

### 核心对象

- `SessionStore`
- `SessionCheckpoint`
- `ResumeSnapshot`
- `ShortTermSessionMemory`
- `session.memory`

### 不属于该模块

- turn evaluation
- gateway control routing
- tool execution
- sandbox policy
- cloud control plane

## Tools

### 职责

`Tools` 负责把外部能力表达成模型可调用、系统可执行、策略可审查的能力单元。

它负责：

- tool definition
- tool registry
- tool execution
- streaming tool execution
- policy engine
- `skill` / `mcp` / `command` 这些工具域内稳定子接口

### 核心对象

- `ToolDefinition`
- `ToolRegistry`
- `ToolExecutor`
- `StreamingToolExecutor`
- `SkillRegistry`
- `McpProtocolClient`
- `McpRuntimeAdapter`
- `CommandModel`

### 不属于该模块

- session transcript
- gateway projection
- sandbox isolation
- background task lifecycle

## Sandbox

### 职责

`Sandbox` 负责执行面的隔离与限制。

它负责：

- execution sandbox
- environment sandbox
- capability model
- security boundary
- execution schema

### 核心对象

- `ExecutionSandbox`
- `EnvironmentSandbox`
- `SandboxCapabilityModel`
- `SecurityBoundary`

### 不属于该模块

- tool definition
- agent turn runtime
- session restore
- cloud control plane scheduling

## Orchestration

### 职责

`Orchestration` 是 cloud 托管 agent 的系统控制面。

它负责：

- cloud control plane lifecycle
- many brains / many hands
- lazy provisioning / wake-resume
- remote hand / execution target routing
- cloud hosting profile 下的职责分布

### 核心对象

- `Orchestrator`
- `ManagedAgentControlPlane`
- `WakeCoordinator`
- `ProvisioningPlanner`
- `HandRouter`
- `HostingProfile`

### 不属于该模块

- chat/session binding
- turn-level model evaluation
- transcript persistence
- tool schema definition
- sandbox execution primitive 本体

## Notes

- `Gateway` 属于 `Harness` 域，不是第六个顶层模块。
- `session.memory` 属于 `Session`，不是顶层 `memory` 模块。
- `command` 是 `Tools` 域内共享对象模型，不是第六个顶层模块。
- host scheduler / periodic ticking 如需存在，通常应归在 host 或 orchestration 侧，而不是 `Gateway` 本体。
- 当前产品关系模型默认是：`1 Agent = 1 Gateway = 1 Global Long-Term Memory`，而 `1 Chat = 1 Session = 1 Short-Term Memory`
- `Harness` 与 `Session` 的 `1:1` 应理解为单活 lease，而不是永久 owner 关系
