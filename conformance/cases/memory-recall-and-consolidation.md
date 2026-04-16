# Case: Memory Recall And Consolidation

## 目标

验证 `memory recall` 与 `memory consolidation` 的分层语义。

## Preconditions

- session / context assembly 支持 memory recall
- session / memory subsystem 支持 memory consolidation 或等价 durable memory update
- 存在一段可被后续 turn 召回的 durable memory

## Ingress

1. 先完成一轮会写入 durable memory 的交互
2. 再发起一轮需要用到该记忆的请求
3. 触发 consolidation 或等价的 memory merge/update 流程

## Expected Lifecycle

- recall 发生在后续 turn 的 context assembly 过程中
- consolidation 不应阻塞普通 turn 的可继续性

## Expected Runtime Semantics

- recalled memory 应进入 context plane，而不是伪装成 transcript 原消息
- consolidation 应更新 durable memory，而不是改写历史 transcript
- short-term memory、durable memory、transcript 三者边界必须清晰

## Expected Persistent Effects

- transcript 追加正常消息
- durable memory store 被新增或更新
- 恢复 session 后，更新后的 durable memory 仍可被后续 recall

## Allowed Variance

- consolidation 可以同步或异步执行
- recall 结果可以进入 structured context 或 attachment-like context
- consolidation 可以失败或延迟，但不能破坏 session restore

## Failure Conditions

- recall 通过篡改 transcript 模拟
- consolidation 直接覆盖历史会话记录
- 恢复后 durable memory 不可继续生效
- consolidation 失败导致 session checkpoint / resume 失效
