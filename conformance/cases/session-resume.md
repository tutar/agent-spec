# Case: Session Resume

## 目标

验证 session durable log 与 restore 语义，而不是仅验证内存消息数组恢复。

## Preconditions

- session 已至少完成一轮 turn
- transcript 或 event log 已 durable 写入
- runtime 支持显式 restore / resume

## Ingress

1. 先完成一轮普通 turn
2. 模拟 host 或 harness 重启
3. 使用同一 session id 恢复
4. 再发送一条新的 user message

## Expected Lifecycle

- 恢复后 session 能重新进入 `running`
- 恢复后新的 turn 能正常完成并回到 `idle`
- 若恢复前处于 `requires_action`，恢复后必须回到同一结构化阻塞语义

## Expected Persistent Effects

- 恢复前消息可被继续引用
- 恢复后新的消息被追加，而不是覆盖旧记录
- 关联 side state 不应无故丢失
- compact boundary、pending action、branch refs 如存在，恢复后仍可追溯

## Host Notes

- `Local` 可以使用本地 durable transcript 或本地数据库
- `Cloud` 可以使用远端 event log store

两者都应满足相同的恢复语义。

## Failure Conditions

- 恢复后 session 被视为新会话
- transcript 顺序丢失或覆盖
- restore 只恢复消息文本，不恢复可继续运行状态
- `requires_action` 只剩下文本提示，缺失原 `tool_use` 绑定
