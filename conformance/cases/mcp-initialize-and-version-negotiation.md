# Case: MCP Initialize And Version Negotiation

## 目标

验证 MCP `initialize -> initialized` 生命周期、protocol version negotiation 与 capability negotiation。

## Preconditions

- runtime 支持 MCP `2025-11-25`
- 至少存在一个可连接的 MCP server

## Ingress

- client 建立 transport
- 发送 `initialize`
- 接收 server 返回的 version / capabilities
- 发送 `notifications/initialized`

## Expected Runtime Behavior

- client 只启用协商成功的 capability
- version 不匹配时，client 必须显式降级或失败，而不是静默继续
- `initialized` 成功后，session handle 可供后续 calls 使用

## Failure Conditions

- 未发送 `initialized`
- 使用未协商 capability
- 忽略 protocol version mismatch
