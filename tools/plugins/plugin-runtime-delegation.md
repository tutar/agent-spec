# Plugin Runtime Delegation

## 职责

本文件定义 loaded plugin 如何把自身 component set 委托给既有子系统，而不是自己执行这些组件。

对 `AgentRuntime` 来说，这一页定义的是 plugin subsystem 在 load 完成后的 delegation surface。

## 稳定接口

推荐最小接口：

```text
PluginDelegationMap
  - command_components -> CommandModel
  - skill_components -> SkillRegistry / SkillInvocationBridge
  - hook_components -> Harness hook runtime
  - mcp_server_components -> McpProtocolClient / McpRuntimeAdapter
  - lsp_server_components -> host LSP integration
  - agent_components -> agent definition loading
  - settings_components -> settings cascade
```

```text
PluginRuntimeDelegator
  - delegate_components(loaded_plugins, runtime_context) -> delegation_result
```

## 委托语义

plugin 不应成为 universal execution engine。

它的职责只是：

- 把 package 中声明的 components 交给既有子系统
- 保留 source identity
- 保留 enablement state
- 保留 component provenance

典型委托关系：

- commands / skills -> command surface / skills
- hooks -> harness hook runtime
- mcp servers -> MCP connection and adaptation
- lsp servers -> host LSP layer
- agents -> agent definition loading
- settings -> settings merge cascade

## 为什么必须独立建模

如果不把 delegation 单独建模，实现者很容易误把 plugin 写成：

- tool wrapper
- command wrapper
- protocol runtime
- mini-orchestrator

这都会混淆边界。

稳定边界应保持：

- `Plugin` 只负责 package 和 source
- capability semantics 仍由 `tool / command / skill / mcp` 等子域定义
- execution 仍由 `ToolExecutor`、`AgentRuntime`、harness hook runtime 或 MCP runtime 承担

## 规范结论

- plugin 的运行时价值是 component packaging + runtime delegation
- plugin 不等于 capability object
- plugin 不应自己承载 tool execution、command invocation 或 MCP protocol session
