# MCP Server Capabilities

## 职责

本文件定义 MCP server 对外暴露能力的最小语义。

当前规范对齐 MCP `2025-11-25`，不再把 server capability 简化成 `tools / prompts / resources` 三类。

## Server Capability View

推荐共享对象：

```text
McpServerCapabilitiesView
  - tools?
  - prompts?
  - resources?
  - logging?
  - completions?
  - tasks?
```

约束：

- capability 未协商即不可调用
- runtime adaptation 层不能假设所有 server 都支持同一能力集

## Tools

最小能力包括：

- `tools/list`
- `tools/call`
- pagination
- `notifications/tools/list_changed`

推荐共享对象：

```text
McpToolDescriptor
  - server_id
  - name
  - title?
  - description?
  - input_schema?
  - annotations?
  - icons?
```

## Prompts

最小能力包括：

- `prompts/list`
- `prompts/get`
- pagination
- `notifications/prompts/list_changed`

推荐共享对象：

```text
McpPromptDescriptor
  - server_id
  - name
  - title?
  - description?
  - arguments?
```

prompt message 可包含 embedded resource；runtime adaptation 不应强行退化为纯文本。

## Resources

最小能力包括：

- `resources/list`
- `resources/read`
- `resources/templates/list`
- `resources/subscribe`
- `resources/unsubscribe`
- `notifications/resources/list_changed`
- `notifications/resources/updated`

推荐共享对象：

```text
McpResourceDescriptor
  - server_id
  - uri
  - name
  - title?
  - description?
  - mime_type?
  - annotations?
  - size?
```

约束：

- `uri` 是资源身份的稳定锚点
- `subscribe` / `updated` 语义属于 resource 观察面，不应被错误映射成 tool result

## Logging

若 server 声明 `logging`，client 应能接收结构化日志通知。

推荐共享对象：

```text
McpLogEvent
  - server_id
  - level
  - logger?
  - data
  - timestamp?
```

## Completions

若 server 声明 `completions`，client 应能为 prompt / tool 参数提供补全请求。

补全结果是辅助能力，不应改变主调用语义。

## Tasks

MCP `2025-11-25` 的 `tasks` 当前属于实验能力，但建议显式保留稳定接口：

```text
McpTaskHandle
  - server_id
  - task_id
  - status
  - operation
  - result_ref?
  - metadata?
```

若声明支持 `tasks`，至少应明确：

- list / cancel / deferred result retrieval
- task status 与普通 request lifecycle 的边界
- task result 如何投影到本地 runtime event
