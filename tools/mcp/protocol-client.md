# MCP Protocol Client

## 职责

本文件定义 MCP client 的协议生命周期。

这里讨论的是 MCP `2025-11-25` base protocol，而不是本地 `Tool` 封装。

对 `AgentRuntime` 来说，这一页定义的是 protocol session lifecycle，而不是本地 capability object model。

## 生命周期

一个合规的 MCP client 至少需要闭合以下链路：

1. 建立 transport
2. 发送 `initialize`
3. 完成 protocol version / capability negotiation
4. 发送 `notifications/initialized`
5. 处理后续 request / notification / progress / cancellation
6. 在宿主退出或 server 不再可用时关闭 session

## 稳定接口

```text
McpProtocolClient
  - initialize(server_descriptor, client_capabilities, client_info) -> McpSessionHandle
  - send_initialized(session_handle)
  - ping(session_handle)
  - cancel_request(session_handle, request_id)
  - close(session_handle)
```

## 最小语义

`initialize` 至少需要承载：

- `protocolVersion`
- client capabilities
- client implementation info

`McpSessionHandle` 应至少记录：

- `server_id`
- `protocol_version`
- `negotiated_capabilities`
- `transport`
- `session_id?`
- `auth_state?`

约束：

- client 只能使用双方都声明过的 capability
- 若 server 返回不同 `protocolVersion`，client 必须明确接受、降级或断开
- `notifications/initialized` 不是可选优化，而是 lifecycle 的一部分
- `cancel_request` 必须与本地 `AgentError(source=transport | tool)` 语义兼容

## 协议级通用能力

一个面向 agent 的 MCP client 还应统一处理：

- ping / health probing
- progress 通知投影
- request timeout
- request cancellation
- graceful shutdown

这些能力不应由单个 `McpToolAdapter` 各自实现。

## Tasks

MCP `2025-11-25` 引入了 `tasks` 能力，目前仍属实验能力，但如果实现声明支持该版规范，建议显式建模。

协议客户端至少应能：

- 识别 `tasks` capability 是否被协商
- 处理 task handle / task status / deferred result retrieval
- 把 task 生命周期映射到本地 `RuntimeEvent` 或等价观测面

## 与其它文档的边界

- transport / auth 见 [transport-and-auth.md](transport-and-auth.md)
- server capability 结构见 [server-capabilities.md](server-capabilities.md)
- client capability 结构见 [client-capabilities.md](client-capabilities.md)
- 本地对象映射见 [runtime-adaptation.md](runtime-adaptation.md)
