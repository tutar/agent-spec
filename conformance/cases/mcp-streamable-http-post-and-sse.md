# Case: MCP Streamable HTTP POST And SSE

## 目标

验证 MCP `Streamable HTTP` transport 在 JSON 与 SSE 两种响应形态下的语义一致性。

## Preconditions

- runtime 支持 MCP HTTP transport
- server 支持 `Streamable HTTP`

## Ingress

- 通过 POST 发送 JSON-RPC request
- 一次使用 JSON response
- 一次使用 `text/event-stream` response

## Expected Runtime Behavior

- 两种 transport response 都保持同样的 request/result 语义
- `MCP-Protocol-Version` header 被正确传递与验证
- SSE reconnect / polling 差异不改变外部可观察结果

## Failure Conditions

- 仅 JSON 可用，SSE 路径语义漂移
- 忽略协议版本 header
- 将 transport 差异泄漏为不同 capability semantics
