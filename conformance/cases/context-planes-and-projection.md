# Case: Context Planes And Projection

## 目标

验证 `Harness.ContextEngineering` 将模型可见输入维持为多平面结构，而不是提前退化成单一 prompt 字符串。

## Preconditions

- runtime 支持 bootstrap prompt、structured context、attachments 与 tool surface 的分层装配
- 至少有一轮 turn 同时使用：
  - bootstrap prompt
  - `system_context`
  - `user_context`
  - attachment
  - tool schema surface

## Ingress

1. 启动一个普通 turn
2. 在同一轮中注入稳定 system skeleton、结构化 `system_context`、结构化 `user_context`
3. 附加一个 message-level attachment
4. 暴露至少一个工具 schema 或语义等价 capability surface

## Expected Runtime Semantics

- 模型可见输入至少区分：
  - `system_prompt`
  - `message_stream`
  - `attachment_stream`
  - `capability_surface`
  - `request_metadata`
- `system_context` 与 `user_context` 不得被建模成同一种注入机制
- tool schemas 或语义等价 capability description 必须属于模型可见上下文的一部分
- attachment 必须保持独立于 transcript 本体的装配语义
- evidence ref 若被使用，必须是上下文引用语义，而不是 transcript 事实本体

## Expected Persistent Effects

- session transcript 或语义等价 durable history 不应因为 context plane 分层而失真
- compact / resume 后仍应保持这些 planes 的边界语义

## Allowed Variance

- 不同实现可以使用不同的模型 API 投影格式
- `request_metadata` 可在 provider 顶层字段、envelope 字段或语义等价层表示

## Failure Conditions

- 所有输入在进入模型前被无差别拼成一个大字符串
- `system_context` 与 `user_context` 无法区分
- tool schema surface 被错误视为 transcript 附件
- attachment 被错误等同于 durable transcript 本体
