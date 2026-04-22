# Tools

## 职责

`Tools` 模块负责为 `AgentRuntime` 提供 capability assembly、command/tool exposure、policy-integrated execution，以及协议接入和扩展包分发能力。

模块总览见 [../module-overview.md](../module-overview.md)，术语归属见 [../terminology-and-ownership.md](../terminology-and-ownership.md)。

对 runtime 来说，这个模块至少闭合三条主线：

- tool definition
- tool registry / capability assembly
- tool execution / streaming execution

同时它还需要提供三类非直接执行面：

- `command`
  共享入口对象模型，承载 prompt/local/review 型入口
- `skill`
  被 runtime 发现、披露、激活并桥接调用的 prompt/workflow capability
- `mcp`
  被 runtime 建立连接、协商能力并适配到本地能力面的协议接入层
- `plugin`
  被 runtime 发现、加载、合并并委托给既有子系统的 capability package layer

## 与 AgentRuntime 的关系

`AgentRuntime` 通过 `Tools` 模块完成：

- startup / refresh 时的 capability source assembly
- turn entry 时的 command / skill / tool surface construction
- tool loop 中的 resolution、policy integration 与 execution handoff
- 协议对象与扩展包的 runtime adaptation / delegation

`Tools` 不负责：

- session state ownership
- turn lifecycle orchestration 本身
- permission host ownership
- plugin installation UI 或 marketplace UX

## 稳定接口

推荐最小接口：

```text
ToolDefinition
  - name
  - input_schema
  - description()
  - call()
  - check_permissions()
  - is_concurrency_safe()

ToolRegistry
  - list_tools()
  - resolve_tool(name)
  - filter_visible_tools(policy, runtime)

ToolExecutor
  - run_tools(tool_calls, context)
```

推荐共享对象：

```text
ToolExecutionHandle
  - execution_id
  - tool_use_ids[]
  - started_at
  - session_id?
  - task_id?
```

```text
PersistedToolResultRef
  - tool_use_id
  - storage_ref
  - preview?
  - original_size?
  - media_type?
```

推荐补充的 shared capability interfaces：

```text
CommandModel
  - resolve_command(name)
  - list_commands(scope)
  - invoke_command(name, args, context)

SkillRegistry
  - discover_skills()
  - load_skill(skill_id)
  - activate_skill(skill_id, context)

McpProtocolClient
  - initialize(server_descriptor)
  - list_tools()
  - list_resources()
  - list_prompts()
  - call_tool()
  - read_resource()
  - get_prompt()

McpRuntimeAdapter
  - adapt_tool()
  - adapt_prompt()
  - adapt_resource()

PluginLoader
  - discover_plugins(scope)
  - load_plugins(runtime_context)
  - merge_plugin_sources(sources)
  - delegate_components(loaded_plugins, runtime_context)
```

## 默认设计取向

- `Tools` 是被 runtime 调用的 capability domain，不是 turn orchestrator
- tool registry 是 capability assembly，不是静态数组
- tool executor 只负责编排调用，不拥有 turn lifecycle
- `tool / skill / mcp / plugin` 必须在能力域内分层建模，再由 runtime 汇合
- `command` 是共享对象模型，不是新的顶层模块
- `plugin` 是 package / source loading / runtime delegation layer，不是执行型 capability
- 协议兼容优先于私有扩展

## 与其它模块的边界

- tool definition 不等于 sandbox
- tool executor 不等于 orchestration
- tool registry 不等于 session history
- skill registry 不等于 tool registry
- mcp client 不等于 mcp runtime adapter
- plugin package 不等于 runtime capability
- command model 不等于顶层模块
- review capability 不等于 Agent Skills

## 阅读结构

### 1. 执行型工具主线

- [tool-model/README.md](tool-model/README.md)
- [tool-model/tool-definition.md](tool-model/tool-definition.md)
- [tool-model/tool-registry.md](tool-model/tool-registry.md)
- [tool-model/policy-engine.md](tool-model/policy-engine.md)
- [../harness/permission/README.md](../harness/permission/README.md)
- [tool-model/tool-executor.md](tool-model/tool-executor.md)
- [tool-model/tool-streaming-execution.md](tool-model/tool-streaming-execution.md)

### 2. 命令与工作流入口主线

- [command-surface/README.md](command-surface/README.md)
- [command-surface/command-model.md](command-surface/command-model.md)
- [command-surface/reflection-and-verification-commands.md](command-surface/reflection-and-verification-commands.md)

### 3. Baseline 与生态接入

- [builtin/README.md](builtin/README.md)
- [builtin/builtin-tool-baseline.md](builtin/builtin-tool-baseline.md)
- [skills/README.md](skills/README.md)
- [mcp/README.md](mcp/README.md)
- [plugins/README.md](plugins/README.md)

这一组回答：

- coding-oriented host 的 builtin baseline 是什么
- skills 如何被导入、披露、激活并进入 runtime
- MCP protocol client 如何协商并接入 server
- plugin package 如何在 runtime startup / refresh 时被发现、加载并委托给既有子系统
