# MCP Runtime Adaptation

## 职责

本文件定义 MCP 协议对象到本地 runtime 能力面的稳定映射。

这里讨论的是适配语义，不是协议本体。

对 `AgentRuntime` 来说，这一页定义的是 capability adaptation surface。

## 默认映射

推荐默认落点：

```text
mcp tools      -> Tool
mcp prompts    -> Command
mcp resources  -> Resource / context surface
mcp logs       -> RuntimeEvent projection
mcp tasks      -> Task / RuntimeEvent projection
```

约束：

- MCP prompt 不应被错误暴露成普通 tool
- resource subscribe / update 不应被错误建模成 tool progress
- logging 是观测面，不是 transcript message
- tasks 是长期运行句柄，不是普通 `tools/call` 的别名

## 稳定接口

```text
McpRuntimeAdapter
  - adapt_tool(server_id, remote_tool)
  - adapt_prompt(server_id, remote_prompt)
  - adapt_resource(server_id, remote_resource)
  - project_log_event(server_id, remote_log_event)
  - project_task_handle(server_id, remote_task_handle)
```

## Source Identity

适配后的本地对象必须保留来源边界。

至少应保留：

- `server_id`
- remote object name / uri
- capability kind
- protocol session or snapshot reference

这样才能保证：

- policy 审查可感知来源
- UI 可区分 builtin / MCP / skill
- conformance 可以检查不同 server 间的边界不被抹平

## Pagination / Refresh / Subscription

runtime adaptation 不应把远端列举结果当作一次性静态数组。

至少需要明确：

- pagination page token 的处理
- list changed notification 后的 refresh 语义
- resource subscribe / unsubscribe 的本地投影

## Error Mapping

MCP protocol error 至少应与本地错误面做以下区分：

- transport failure
- authorization failure
- protocol violation
- capability unavailable
- remote tool execution failure

这些错误不应被统一压缩成普通 `tool_failed` 文本消息。
