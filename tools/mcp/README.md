# MCP

## 职责

`MCP` 子模块负责定义 Model Context Protocol 接入的稳定接口。

它的职责不是“提供一个 MCP tool”，而是：

- 建立并维护 MCP client session
- 完成 protocol version / capability negotiation
- 通过 `stdio` 或 `Streamable HTTP` 连接 MCP server
- 处理 HTTP transport 下的 authorization / discovery
- 枚举和调用 server capabilities
- 在宿主支持时向 server 暴露 client capabilities
- 把 MCP 协议对象适配到本地 runtime 的 `tool / command / resource` 模型

当前规范默认对齐 MCP specification `2025-11-25`。参考官方规范：

- <https://modelcontextprotocol.io/specification/2025-11-25/basic>
- <https://modelcontextprotocol.io/specification/2025-11-25/server>
- <https://modelcontextprotocol.io/specification/2025-11-25/client>

## 分层

`tools/mcp` 应分成三层，而不是把协议层和本地约定混写：

- 协议客户端层
  见 [protocol-client.md](protocol-client.md)
- runtime 适配层
  见 [runtime-adaptation.md](runtime-adaptation.md)
- host 扩展层
  见 [host-extensions.md](host-extensions.md)

这里必须明确：

- MCP core capability 是协议定义的能力
- runtime adaptation 是宿主把协议对象映射到本地 `Tool / Command / Resource`
- `mcp skill`、bundle host、UI affordance 属于 host-specific extension，不属于 MCP core

## 阅读结构

- [protocol-client.md](protocol-client.md)
  协议生命周期、version negotiation、capability negotiation、ping/cancel/progress、shutdown
- [server-capabilities.md](server-capabilities.md)
  `tools / prompts / resources / logging / completions / tasks`
- [client-capabilities.md](client-capabilities.md)
  `roots / sampling / elicitation / tasks`
- [transport-and-auth.md](transport-and-auth.md)
  `stdio`、`Streamable HTTP`、session continuity、OAuth 2.1 / OIDC discovery
- [runtime-adaptation.md](runtime-adaptation.md)
  MCP objects 到本地 `tool / command / resource` 的稳定映射
- [host-extensions.md](host-extensions.md)
  `mcp skill`、MCPB、UI / trust / install-specific affordance

## 稳定接口

推荐最小接口：

```text
McpProtocolClient
  - initialize(server_descriptor, client_capabilities, client_info)
  - send_initialized(session_handle)
  - ping(session_handle)
  - close(session_handle)

McpTransportAdapter
  - open_stdio(config)
  - open_http(config)
  - send(session_handle, message)
  - receive(session_handle)
  - close(session_handle)

McpAuthorizationAdapter
  - discover_authorization(server_endpoint)
  - acquire_token(auth_metadata, scopes?)
  - refresh_token(token_state)
  - handle_www_authenticate(challenge, auth_state)

McpRuntimeAdapter
  - adapt_tool(server_id, remote_tool)
  - adapt_prompt(server_id, remote_prompt)
  - adapt_resource(server_id, remote_resource)
```

若宿主支持 MCP client features，还应补：

```text
McpRootsProvider
  - list_roots()
  - notify_roots_changed()

McpSamplingBridge
  - handle_sampling_request(server_id, request)

McpElicitationBridge
  - handle_elicitation_request(server_id, request)
```

## 默认实现取向

默认实现通常会把 MCP 分成三条链路：

- protocol client
- runtime adaptation
- host extension

实现者可以按宿主需求把这三条链路装配为同进程组件、独立服务或混合模式，但不应改变它们的角色边界。

其中 `mcp skill`、bundle host、安装与信任流程等能力应始终视为 host-specific extension，而不是 MCP `2025-11-25` core requirement。

## 要解决的问题

- 如何把 MCP 协议接入和本地 `tool / command / resource` 抽象解耦
- 如何同时支持 lifecycle、transport、authorization、server features、client features
- 如何支持 `tools / prompts / resources` 之外的 `logging / completions / tasks`
- 如何让 `roots / sampling / elicitation` 在宿主支持时稳定暴露
- 如何在保留来源边界的前提下把 MCP objects 接入本地 runtime
- 如何避免把 host 私有扩展错误提升成 MCP core 合规要求

## 规范结论

- `MCP` 应在 `tools` 域下保留为独立稳定子接口
- 不能把 “MCP 兼容” 简化成 “支持一批远端 tools”
- 必须区分协议客户端层、runtime 适配层和 host 扩展层
- `mcp skill` 发现、MCPB、UI-specific affordance 可以存在，但必须明确标为 host extension
