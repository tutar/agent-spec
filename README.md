# Agent SDK Spec

本目录是一套面向多语言 Agent SDK 实现者的规范包。

它不是 `cc` 源码的逐文件讲解，也不是某个 TypeScript SDK 的 API 参考，而是从当前 `cc` 的本地 agent harness 中抽取出的稳定模块边界、共享对象模型和一致性要求。

本规范默认面向 `Local-first` 宿主形态：

- 单机运行
- 模块可以 direct-call
- 本地 session / tool / sandbox / task 为默认落点

同时要求所有核心接口保持 `Cloud-compatible`：

- transport-neutral
- 可远程化
- 可替换部署位置
- 不把本地实现细节固化为接口前提

本目录按 5 个核心模块组织，每个模块目录都以一个 `README.md` 作为主规范，统一回答四件事：

- 职责
- 稳定接口
- 默认实现
- 要解决的问题

目标不是沉淀某个具体语言或某一版 harness 的实现细节，而是形成一套可向下指导多语言 SDK、向上承接不同模型与基础设施能力的稳定接口规范。

## 规范定位

本规范包的主要交付包括：

- 五个核心模块的稳定语义边界
- 跨模块共享的 canonical object model
- `Local-first` 默认实现映射
- 核心能力与标准扩展能力的分层
- 跨语言 conformance 规则与 golden cases

其中：

- `核心能力`
  宣称实现某个模块时必须保持一致的最小语义
- `标准扩展`
  推荐沉淀的高级能力，如 prompt cache、memory consolidation、background agent；允许采用不同技术方案，但行为语义必须兼容

默认参考实现映射来自当前仓库中的 `cc/` 源码，但这些映射只用于解释默认落点，不构成 SDK API 设计约束。

## 阅读入口

建议先按以下顺序阅读：

1. [module-overview.md](module-overview.md)
2. [terminology-and-ownership.md](terminology-and-ownership.md)
3. 五个核心模块的 `README.md`

## 五个核心模块

- [harness/README.md](harness/README.md)
- [session/README.md](session/README.md)
- [tools/README.md](tools/README.md)
- [sandbox/README.md](sandbox/README.md)
- [orchestration/README.md](orchestration/README.md)

其中 `tools/` 现按职责进一步拆为：

- `tool-model/`
- `command-surface/`
- `builtin/`
- `skills/`
- `mcp/`

其中 [harness/README.md](harness/README.md) 已进一步按 5 组子主题组织：

- Runtime Core
- Model Provider
- Context Assembly
- Gateway
- Extension And Projection

在 `Context Assembly` 子主题下，现已单独补充 bootstrap prompt 规范，用于稳定 system prompt skeleton、section cache 与 static/dynamic boundary。

另外，反思相关能力已按职责拆入现有模块，而不是新增顶层 `reflection` 模块：

- `orchestration` 下的 verification / reflection task
- `harness` 下的 post-turn processing
- `session` 下的 memory consolidation

此外，规范现在显式面向两种宿主场景：

- `Local`
  单机部署，模块可 direct-call，本地 session / task / verifier 为主
- `Cloud`
  托管 control plane + remote execution

这两种场景不改变五个核心模块的边界，但会改变它们的部署位置、职责分布和默认实现映射。相关差异主要通过 `orchestration/local/` 与 `orchestration/cloud/` 表达。

## 共享文档

- [module-overview.md](module-overview.md)
- [terminology-and-ownership.md](terminology-and-ownership.md)
- [conformance.md](conformance.md)
- [conformance/README.md](conformance/README.md)
- [capability-surface.md](capability-surface.md)
- [extension-packaging.md](extension-packaging.md)
- [object-model.md](object-model.md)
- [schema-serialization.md](schema-serialization.md)
- [test-artifacts.md](test-artifacts.md)
- [managed-agent-conformance-scenarios.md](managed-agent-conformance-scenarios.md)
- [todo.md](todo.md)

## 设计原则

- 本规范首先稳定模块边界，再允许实现自由演进。
- 优先稳定 `Harness / Session / Tools / Sandbox / Orchestration` 五个对象。
- 优先复用模型原生能力，能力不足时由 harness 补齐。
- 默认以 `Local-first` 方式抽取语义，再要求核心接口保持 `Cloud-compatible`。
- durable history、执行环境、工具编排、托管编排必须解耦。
- 同一套模块边界必须同时适配 Local、Cloud 两种宿主形态。
- 文档中的接口描述是跨语言语义，不是具体语言 API。
- `command` 会作为 `tools` 域内共享抽象出现，但不升级为第六个顶层模块。
- 当前流行优化方案应优先沉淀为标准扩展，而不是直接绑死某一种实现算法。

## 与当前代码的默认映射

以下映射仅用于说明当前 `cc` 的默认实现如何落到规范边界上。

它们是参考实现，不是要求各语言 SDK 复制的目录结构或类名。

- Harness: [QueryEngine.ts](../cc/QueryEngine.ts), [query.ts](../cc/query.ts), [context.ts](../cc/context.ts)
  Harness 侧还包含入口网关抽象，当前默认映射到 [bridge/initReplBridge.ts](../cc/bridge/initReplBridge.ts), [bridge/replBridge.ts](../cc/bridge/replBridge.ts), [bridge/bridgeMessaging.ts](../cc/bridge/bridgeMessaging.ts), [bridge/replBridgeTransport.ts](../cc/bridge/replBridgeTransport.ts)
- Session: [bootstrap/state.ts](../cc/bootstrap/state.ts), [utils/sessionStorage.ts](../cc/utils/sessionStorage.ts), [utils/sessionRestore.ts](../cc/utils/sessionRestore.ts), [utils/sessionState.ts](../cc/utils/sessionState.ts)
- Tools: [Tool.ts](../cc/Tool.ts), [tools.ts](../cc/tools.ts), [services/tools/toolOrchestration.ts](../cc/services/tools/toolOrchestration.ts), [services/tools/StreamingToolExecutor.ts](../cc/services/tools/StreamingToolExecutor.ts)
  `tools` 域下还包含 `skills`、`mcp` 两个稳定子接口，以及 `command` 共享对象模型；默认映射分别位于 [skills/loadSkillsDir.ts](../cc/skills/loadSkillsDir.ts)、[services/mcp/client.ts](../cc/services/mcp/client.ts)、[types/command.ts](../cc/types/command.ts)
- Sandbox: [tools/BashTool/BashTool.tsx](../cc/tools/BashTool/BashTool.tsx), [tasks/LocalShellTask/LocalShellTask.tsx](../cc/tasks/LocalShellTask/LocalShellTask.tsx), [utils/sandbox/sandbox-adapter.ts](../cc/utils/sandbox/sandbox-adapter.ts)
- Orchestration: [tasks.ts](../cc/tasks.ts), [tasks/LocalAgentTask/LocalAgentTask.tsx](../cc/tasks/LocalAgentTask/LocalAgentTask.tsx), [tasks/RemoteAgentTask/RemoteAgentTask.tsx](../cc/tasks/RemoteAgentTask/RemoteAgentTask.tsx), [tools/AgentTool/runAgent.ts](../cc/tools/AgentTool/runAgent.ts)
  `orchestration` 域下默认还包含标准 agent 编排模式，映射到 [tools/AgentTool/AgentTool.tsx](../cc/tools/AgentTool/AgentTool.tsx)、[tasks/InProcessTeammateTask/InProcessTeammateTask.tsx](../cc/tasks/InProcessTeammateTask/InProcessTeammateTask.tsx)、[tools/shared/spawnMultiAgent.ts](../cc/tools/shared/spawnMultiAgent.ts)

## 关于源码分析材料

本规范以当前仓库中可见的 `agent-sdk-spec/` 和 `cc/` 源码为权威来源。

如果其他说明文档或 README 引用了当前快照中不存在的分析材料，应将其视为背景说明，而不是规范前提。规范内容必须能够在不依赖缺失文档的前提下独立成立。
