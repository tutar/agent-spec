# Tool Streaming Execution

## 目标

本文档定义工具的增量执行模式。

对 `AgentRuntime` 来说，`StreamingToolExecutor` 是在流式采样期间被调用的增量执行接口，而不是新的 turn orchestrator。

这里的 streaming execution 指：

- tool_use 在 assistant 流中逐步出现
- executor 在完整 turn 结束前就启动工具
- progress 与 result 以增量方式回流

它不等于模型流式协议，也不等于普通批量工具编排。

## 一、职责

`StreamingToolExecutor` 负责：

- 增量接收 `tool_use`
- 尽早启动可执行工具
- 发射 progress
- 回收最终 result
- 在 fallback / abort / sibling error 下进行一致性回收

## 二、为什么需要单独建模

批量工具编排只能在完整 assistant turn 结束后启动。

但在流式模型返回下，tool_use 可能很早就已确定。

如果等完整 assistant turn 结束再启动工具，会损失：

- 并行时间
- 用户可见进度
- 长任务提前启动能力

因此需要增量执行模式。

## 三、推荐最小接口

```text
StreamingToolExecutor
  - add_tool(tool_use, assistant_message_ref)
  - get_completed_results() -> execution_updates
  - get_remaining_results() -> execution_updates
  - discard()
```

其中 `execution_updates` 推荐至少包括：

```text
ToolExecutionUpdate
  - progress_message?
  - result_message?
  - context_modifier?
  - persisted_ref?
```

## 四、必须支持的能力

- 增量接收 `tool_use`
- 基于并发安全性启动执行
- progress 与 result 分离
- 结构化 abort reason
- sibling error 传播
- streaming fallback 后 discard
- 保持结果发射顺序可控
- 与 batch executor 共享结果持久化和 context modifier commit 语义

## 五、Abort Reason

增量工具执行的取消不应只是布尔值。

推荐至少支持：

- `sibling_error`
- `user_interrupted`
- `streaming_fallback`

这样 executor 才能决定：

- 是否生成 synthetic error result
- 是否中止兄弟工具
- 是否放弃未完成结果

## 六、与批量 ToolExecutor 的关系

- `ToolExecutor`
  解决批量工具编排
- `StreamingToolExecutor`
  解决增量工具执行

推荐关系：

- streaming 模式下优先使用增量执行器
- non-streaming 或 turn-end batch 模式下使用普通执行器
- 两者共享同一套 tool definition、权限和 context mutation 语义
- 两者必须共享同一套 terminal state、error class、persisted result 和 context commit 语义

## 七、与 AgentRuntime 的关系

`AgentRuntime` 负责：

- 决定何时创建新的 streaming executor
- 在 fallback 时 tombstone 旧消息并 discard 旧 executor
- 将 executor 回流的结果重新接回消息链

`StreamingToolExecutor` 不负责：

- turn continuation
- tombstone assistant message
- transcript persistence

## 八、默认实现映射

本仓库当前默认实现映射到：


关联批量执行实现：


默认实现特征：

- 支持 incremental add
- 按 `isConcurrencySafe` 决定并发
- progress 单独缓存并尽快发射
- streaming fallback 时 `discard()`
- sibling error 可传播取消

## 九、规范结论

- 工具增量执行应成为独立规范概念
- progress、result、abort reason 必须显式建模
- batch execution 与 streaming execution 应共享语义，但不应强行合并为同一种执行器
- `streaming_fallback`、`user_interrupted`、`sibling_error` 等 abort reason 应能映射到统一 terminal/error 语义，见 [../../harness/runtime/core/failure-and-terminal-states.md](../../harness/runtime/core/failure-and-terminal-states.md)
- streaming executor 不得发明与 batch executor 不一致的 persisted result 或 context commit 语义
