# Harness

## 职责

`Harness` 是直接围绕模型运行的一层控制逻辑。

模块总览见 [../module-overview.md](../module-overview.md)，术语归属见 [../terminology-and-ownership.md](../terminology-and-ownership.md)。

它负责：

- 读取 session 中的可用上下文
- 根据模型能力选择 native path 或 fallback path
- 组装 prompt / messages / tool surface
- 驱动一轮或多轮 agent loop
- 把模型输出路由到 tools、session 和 orchestration

它必须对宿主形态保持中立：

- 在 `Local` 场景下，通常作为本地 turn runner
- 在 `Cloud` 场景下，通常作为可横向扩展的远端 turn runner

本规范中的 `Harness` 采用 `Local-first` 抽取方式：

- 默认实现通常与本地 session、tool registry、execution sandbox 同机协作
- 但稳定接口不得要求同机、同进程或同网络边界
- 所有 direct-call 优化都应被视为实现细节，而不是接口语义

## 稳定接口

推荐最小接口：

```text
Harness
  - run_turn(input, session_handle) -> runtime_events, terminal_state
  - build_model_input(session_slice, context_providers) -> model_input
  - handle_model_output(output) -> next_actions
  - route_tool_call(tool_call) -> tool_result
  - emit_runtime_event(event) -> projected_event
```

这些接口是跨语言语义契约，而不是某个语言 SDK 中必须逐字出现的方法名。

### 核心能力

宣称实现 `Harness` 模块时，至少应具备：

- turn / interaction 推进能力
- model input 组装能力
- model capability routing
- runtime event 发射
- tool continuation routing
- terminal state 收敛

### 标准扩展

推荐把下列能力作为 `Harness` 标准扩展实现：

- `ContextGovernance`
  - context budget
  - overflow recovery
  - auto compact
  - budget continuation
- `PromptCacheStrategy`
  - stable prefix
  - dynamic suffix
  - fork sharing
  - cache break detection
  - strategy equivalence
- `Streaming`
  - assistant delta
  - tool-use streaming continuation
  - withheld error / recovery
- `PostTurnProcessing`
  - verification
  - reflection
  - summarization
  - observability projection

这些扩展可以采用不同算法和底层技术，但必须保持触发时机、外部可观察行为和恢复语义一致。

## 默认实现

当前代码库中的默认 harness 由以下几层拼起来：

- [QueryEngine.ts](../../cc/QueryEngine.ts)
  负责会话级输入处理、消息持有、transcript 落盘、query 驱动
- [query.ts](../../cc/query.ts)
  负责核心 agent loop、tool use 回路、budget/compact/recovery
- [context.ts](../../cc/context.ts)
  负责 system/user context 提供

默认路径是：

1. `QueryEngine` 接收用户输入
2. 先写入 transcript，保证 session 可恢复
3. 调 `query()` 推进 turn
4. `query()` 在模型返回 tool use 时调用 tool executor
5. 结果回填消息链，继续下一轮或结束

这里的 `QueryEngine + query()` 只是当前本地默认实现。

各语言 SDK 可以选择同步循环、异步状态机、actor model 或 service-oriented runner，只要对外保持同等 turn 语义即可。

## 要解决的问题

- 如何把一次模型请求扩展成多轮 agent 事务
- 如何在 tool use、context compact、fallback 之间保持语义连续
- 如何把 prompt cache、streaming、post-turn processing 等优化沉淀为可替换扩展
- 如何让模型能力演进时，harness 可以改变而接口不必一起改变
- 如何让 harness 崩溃时，session 仍然可被新 harness 接管
- 如何在 Local、Cloud 两种宿主形态下保持同一 turn 语义

## 与其它模块的边界

- 不负责 durable session storage
- 不负责 hand provisioning
- 不直接持有凭据
- 不等于 orchestration 控制平面
- 不应绑定某一个特定 host（终端、GUI 或云端服务）

## 目录内文档

### 0. Deployment Boundaries

- [deployment-boundaries.md](deployment-boundaries.md)

这一页回答：

- 同进程优化和分布式语义应如何区分
- 哪些接口必须保持 transport-neutral
- 为什么 `Cloud` 不应被实现成 “本地 runtime + 远程工具”

### 1. Runtime Core

- [runtime-overview.md](runtime-core/runtime-overview.md)
- [agent-runtime.md](runtime-core/agent-runtime.md)
- [failure-and-terminal-states.md](runtime-core/failure-and-terminal-states.md)
- [state-layering.md](runtime-core/state-layering.md)
- [message-and-event-pipeline.md](runtime-core/message-and-event-pipeline.md)

这一组回答：

- harness 主循环是什么
- turn 如何推进
- 失败、取消、阻塞和终止态如何统一
- 内部状态怎么分层
- 内部消息与外部事件如何分开

### 2. Model Provider

- [model-capability-routing.md](model-provider/model-capability-routing.md)
- [model-provider-adapter.md](model-provider/model-provider-adapter.md)
- [model-provider-streaming-adapter.md](model-provider/model-provider-streaming-adapter.md)

这一组回答：

- 不同模型能力如何探测与降级
- 模型与 provider 协议差异如何吸收
- 模型 streaming 如何接入 harness
- provider transport / auth / endpoint 边界如何落在统一 adapter 内

### 3. Context Assembly

- [bootstrap-prompts.md](context-assembly/bootstrap-prompts.md)
- [context-provider.md](context-assembly/context-provider.md)
- [context-governance.md](context-assembly/context-governance.md)
- [prompt-cache-strategy.md](context-assembly/prompt-cache-strategy.md)

这一组回答：

- bootstrap prompt 骨架如何构建
- 上下文如何分层装配
- memory / attachment / delta 如何进入模型输入
- 上下文预算、compact、治理如何实现
- Anthropic-native 与 OpenClaw-mediated prompt cache 策略如何挂到 harness

### 4. Gateway

- [gateway.md](gateway/gateway.md)
- [channel-adapter.md](gateway/channel-adapter.md)
- [interaction-loop.md](gateway/interaction-loop.md)
- [bridge-transport.md](gateway/bridge-transport.md)
- [bridge-message-projection.md](gateway/bridge-message-projection.md)
- [local-session-adapter.md](gateway/local-session-adapter.md)

这一组回答：

- 外部 channel / chat 如何接入系统
- channel adapter 应落在哪一侧
- gateway 与 harness 之间的交互循环如何定义
- bridge transport 如何抽象
- 内外消息如何投影
- 本地 runtime 如何适配成 gateway 可承载 session

### 5. Extension And Projection

- [agent-observability.md](extension-and-projection/agent-observability.md)
- [hook-and-lifecycle-extensions.md](extension-and-projection/hook-and-lifecycle-extensions.md)
- [post-turn-processing.md](extension-and-projection/post-turn-processing.md)
- [sdk-event-transport.md](extension-and-projection/sdk-event-transport.md)

这一组回答：

- agent 运行状态、进度、usage、trace 如何投影
- lifecycle hook 如何扩展 harness
- turn 结束后的后处理如何触发和编排
- runtime state 如何投影给外部 SDK client

## 规范结论

- harness 是“脑”，但不是整个系统
- harness 应尽量无状态，或仅保留可从 session 重建的短状态
- harness 应优先依赖稳定接口，而不是假设自己与 hands 共机
- harness 的稳定接口应先满足跨进程/跨部署边界语义，再允许提供同进程优化 binding
- harness 相关状态应至少区分 `ProcessState`、`InteractionState` 与各领域状态投影
- harness 规范必须同时适配 Local、Cloud 两种 host
- prompt cache、context governance、streaming 与 post-turn processing 应优先建模为标准扩展，而不是隐含在某个默认实现内部
