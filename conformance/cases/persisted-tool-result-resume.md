# Case: Persisted Tool Result Resume

## 目标

验证工具大结果外存化后，在 compact / resume / branch restore 下仍保持稳定引用语义。

## Preconditions

- 至少一个工具会产生超限结果
- runtime 支持 `PersistedToolResultRef` 或语义等价对象
- session 支持 compact 与 resume

## Ingress

1. 触发一个会生成大结果的工具调用
2. 确认该结果被外存化
3. 触发 compact 或等价 context governance
4. 模拟 session resume

## Expected Runtime Semantics

- `tool_result` 中出现稳定引用
- compact 后不会要求重新生成原始结果
- resume 后同一 `tool_use_id` 仍映射到同一 persisted ref 语义

## Failure Conditions

- compact 后引用丢失
- resume 后 persisted ref 漂移到不同外部结果
- 需要重跑原工具才能恢复同一结果引用
