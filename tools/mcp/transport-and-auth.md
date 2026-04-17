# MCP Transport And Authorization

## 职责

本文件定义 MCP `2025-11-25` transport 与 authorization 的稳定要求。

## Transport

推荐接口：

```text
McpTransportAdapter
  - open_stdio(config) -> transport_handle
  - open_http(config) -> transport_handle
  - send(session_handle, message)
  - receive(session_handle)
  - resume(session_handle, stream_ref?)
  - close(session_handle)
```

### stdio

`stdio` transport 的最小语义：

- 基于 JSON-RPC message stream
- server `stdout` 只应用于 MCP messages
- `stderr` 可用于日志
- 连接生命周期绑定本地进程生命周期

### Streamable HTTP

`Streamable HTTP` transport 的最小语义：

- 单 endpoint 支持 GET / POST
- POST 发送 JSON-RPC request
- `Accept` 至少兼容 `application/json` 与 `text/event-stream`
- response 可为单个 JSON，也可为 SSE stream
- 必须保留 `MCP-Protocol-Version` header 语义
- 需要处理 reconnection / polling / redelivery / session continuity

约束：

- transport 差异不应改变外部可观察的 capability semantics
- SSE / polling 差异可以存在，但 request / notification 的语义不能漂移

## Authorization

HTTP transport 下，推荐接口：

```text
McpAuthorizationAdapter
  - discover_authorization(server_endpoint) -> auth_metadata
  - acquire_token(auth_metadata, scopes?) -> token_state
  - refresh_token(token_state) -> token_state
  - handle_www_authenticate(challenge, auth_state) -> auth_state
```

最小要求：

- 支持 OAuth 2.1 语义
- 支持 resource metadata / authorization server discovery
- 支持 OIDC discovery
- 支持 `WWW-Authenticate` challenge 带来的增量 scope 升级

约束：

- HTTP auth 规范不应错误套用到 `stdio`
- auth discovery 失败必须明确映射成 `AgentError(source=transport | auth)`
- session continuity 与 token state 应分离，不应把 token 刷新当成新 MCP session

## Security

agent 规范至少应明确：

- HTTP origin validation
- localhost / loopback binding guidance
- auth boundary 与 trust boundary 的分离
- 不能因宿主支持远端 MCP 就绕过本地 policy / sandbox 审查
