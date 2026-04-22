# MCP

## 职责

`MCP` 子模块负责定义 Model Context Protocol 接入的稳定接口。

对 `AgentRuntime` 来说，这一组接口回答的是：

- startup / refresh 阶段如何建立和维护 protocol session
- 如何把远端 capability 适配到本地 `tool / command / resource` 面
- 哪些能力属于 MCP core，哪些只属于 host extension

## 分层

- 协议客户端层
  见 [protocol-client.md](protocol-client.md)
- runtime 适配层
  见 [runtime-adaptation.md](runtime-adaptation.md)
- host 扩展层
  见 [host-extensions.md](host-extensions.md)

## 阅读结构

- [protocol-client.md](protocol-client.md)
- [server-capabilities.md](server-capabilities.md)
- [client-capabilities.md](client-capabilities.md)
- [transport-and-auth.md](transport-and-auth.md)
- [runtime-adaptation.md](runtime-adaptation.md)
- [host-extensions.md](host-extensions.md)

## 规范结论

- `MCP` 是独立稳定子接口，不应被简化成“支持一批远端 tools”
- runtime adaptation 只负责把协议对象映射到本地能力面
- `mcp skill`、bundle host、UI / trust / install affordance 必须明确标为 host extension
