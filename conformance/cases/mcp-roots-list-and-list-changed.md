# Case: MCP Roots List And List Changed

## 目标

验证 client capability `roots` 的列举与变更通知语义。

## Preconditions

- client 声明支持 `roots`
- server 会调用 `roots/list`

## Ingress

- server 请求 `roots/list`
- client 返回 roots
- root 集合变化后发出 `roots/list_changed`

## Expected Runtime Behavior

- roots 使用稳定 `file://` URI 语义
- 变更通知后，server 可观察到新的 roots 列表

## Failure Conditions

- roots 不是稳定 URI
- roots 变化未投影为 notification
