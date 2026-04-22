# MCP Client Capabilities

## 职责

本文件定义宿主作为 MCP client 向 server 暴露的能力面。

当前规范对齐 MCP `2025-11-25` 的 client features。

对 `AgentRuntime` 来说，这一页定义的是 protocol session 可向远端暴露的 client capability surface。

## Client Capability View

推荐共享对象：

```text
McpClientCapabilities
  - roots?
  - sampling?
  - elicitation?
  - tasks?
  - experimental?
```

## Roots

若宿主声明 `roots` capability，应至少支持：

- `roots/list`
- `notifications/roots/list_changed`

推荐接口：

```text
McpRootsProvider
  - list_roots() -> McpRoot[]
  - notify_roots_changed()
```

推荐共享对象：

```text
McpRoot
  - uri
  - name?
  - writable?
```

约束：

- `roots` 应使用稳定 `file://` URI 语义
- roots 变化必须可投影到 notification，而不是只更新本地缓存

## Sampling

若宿主声明 `sampling` capability，应能处理 server 发起的 sampling request。

推荐接口：

```text
McpSamplingBridge
  - handle_sampling_request(server_id, request) -> result
```

推荐共享对象：

```text
McpSamplingRequest
  - server_id
  - request_id
  - model_preferences?
  - messages
  - system_prompt?
  - tools?
  - tool_choice?
  - metadata?
```

约束：

- sampling request 必须可进入 user-in-the-loop 审批或 host policy 审查
- 若宿主不支持 `tools` / `toolChoice` in sampling，则不得在 capability negotiation 后谎报支持

## Elicitation

若宿主声明 `elicitation` capability，应能处理结构化用户输入请求。

推荐接口：

```text
McpElicitationBridge
  - handle_elicitation_request(server_id, request) -> result
```

推荐共享对象：

```text
McpElicitationRequest
  - server_id
  - request_id
  - mode: form | url
  - title?
  - schema?
  - url?
  - explanation?
```

约束：

- 敏感信息收集应优先走 `url` mode，而不是普通表单 mode
- elicitation 的结果应能映射回本地 `RequiresAction(user_input)` 或等价对象

## Tasks

若宿主声明 client-side `tasks` 支持，应明确：

- task-augmented sampling / elicitation request 的处理语义
- deferred completion 与 foreground request 的边界
- task cancellation 如何映射回本地 runtime
