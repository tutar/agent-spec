# Runtime Overview

## 目标

agent runtime 的核心目标是管理一轮完整的 agentic turn，而不是只包装一次模型调用。

这里的 agent 是跨语言的 agent harness。它不预设具体编程语言，也不预设底层只有一种模型能力形态。

更准确地说，这是一套 `meta-harness` 规范：它希望稳定的是 agent 周围的接口，而不是某一代 harness 自己的实现细节。

一轮 turn 至少包含：

1. 组装上下文
2. 发起模型请求
3. 流式接收 assistant 内容
4. 解析并执行 tool use
5. 回填 tool result
6. 根据预算、权限和状态决定继续、阻塞、压缩或结束

## 标准组件

- `ModelProviderAdapter`
  负责适配不同模型和 provider 的协议与能力声明。
- `Session`
  负责 durable event log、恢复与会话标识。
- `Harness`
  负责调用模型、塑形上下文、路由工具与推进 agent loop。
- `HarnessInstance`
  负责承载某个 session-bound harness worker 的实例身份与状态。
- `Sandbox / Hands`
  负责执行代码、编辑文件或访问外部资源的可替换执行面。
- `ContextProvider`
  负责提供系统、用户、记忆、附件等上下文来源。
- `AgentRuntime`
  负责 `HarnessInstance` 内部的 turn 级状态机和消息推进。
- `ToolRegistry`
  负责能力装配和工具可见性控制。
- `ToolExecutor`
  负责工具调用编排、取消、结果发射和上下文修改串行化。
- `TaskManager`
  负责 harness 管理的后台任务、子 agent 和其他长生命周期执行对象。
- `PolicyEngine`
  负责规则、审批和 `requires_action`。
- `ContextGovernance`
  负责 token budget、compact 和大结果外存化。

## 推荐数据流

```text
User/Input
  -> Model Capability Routing
  -> Session
  -> Harness
  -> ContextProvider
  -> AgentRuntime
  -> Model Stream
  -> ToolExecutor
  -> Sandbox / Hands
  -> PolicyEngine
  -> ContextGovernance
  -> AgentRuntime Terminal State
```

## 标准事件模型

agent 应统一对外暴露事件流，而不是只返回最终文本。

建议最小事件集：

- `request_started`
- `assistant_delta`
- `assistant_message`
- `tool_started`
- `tool_progress`
- `tool_result`
- `requires_action`
- `context_compacted`
- `turn_completed`
- `turn_failed`

## 默认约束

- runtime 必须兼容不同模型的能力差异。
- runtime 必须优先复用模型原生能力。
- runtime 必须在模型能力不足时提供 harness fallback。
- session、harness、sandbox 必须允许独立替换与独立故障恢复。
- session、tool executor、sandbox 的稳定接口不应默认建立在同进程调用语义上。
- runtime 必须支持多轮 tool loop。
- runtime 必须支持流式工具执行。
- runtime 必须支持中断和恢复。
- runtime 必须支持权限阻塞。
- runtime 必须支持长会话上下文治理。
- gateway 若管理多个并发 worker，应把它们建模为多个 `HarnessInstance`，而不是多个顶层 `AgentRuntime`

## 当前仓库映射


## 规范结论

- 规范的核心不只是 runtime 组件，而是顶层接口对象
- `session / harness / sandbox` 应被视为长期稳定接口
- runtime 是跨语言 harness 的中心
- runtime 负责统一上层语义，而不是暴露底层模型差异
- 模型适配与 runtime 编排必须解耦
- deployment boundary 只应体现在 binding 层，不应污染核心运行时语义
