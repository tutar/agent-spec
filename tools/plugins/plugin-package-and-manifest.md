# Plugin Package And Manifest

## 职责

本文件定义 plugin package 的最小对象模型，以及 manifest 需要表达的稳定语义。

对 `AgentRuntime` 来说，这一页定义的是 package metadata 和 component declaration，而不是执行型 capability contract。

## 稳定对象

推荐最小对象：

```text
PluginManifest
  - name
  - version?
  - description?
  - source?
  - commands?
  - skills?
  - agents?
  - hooks?
  - mcp_servers?
  - lsp_servers?
  - settings?
  - user_config?
  - output_styles?
```

```text
LoadedPlugin
  - plugin_id
  - manifest
  - source
  - path?
  - enabled
  - builtin?
  - component_set
```

```text
PluginComponentSet
  - command_components[]
  - skill_components[]
  - agent_components[]
  - hook_components[]
  - mcp_server_components[]
  - lsp_server_components[]
  - settings_components[]
  - output_style_components[]
```

## 语义要求

- plugin 不是 runtime capability object
- plugin 的价值在于把多类 component 打包成单一 source unit
- manifest 至少需要表达 source identity、component declaration 和 enablement-affecting metadata
- `LoadedPlugin` 必须可区分 builtin / installed / inline source family
- manifest 可以使用文件路径、内联描述或语义等价对象表达 component declaration

## 默认实现映射

本规范允许本地文件系统实现把包内 manifest 放在：

- `.openagent/plugin.json`

但该路径只是默认实现映射，不是跨语言强约束。

规范真正稳定的是：

- package has a manifest
- manifest declares a component set
- runtime can materialize a `LoadedPlugin`

## 与其它子域的边界

- `Plugin` 不定义 `ToolDefinition`
- `Plugin` 不定义 `CommandModel`
- `Plugin` 不定义 `SkillDefinition`
- `Plugin` 不定义 MCP protocol session

这些能力都由各自子域定义；plugin 只负责 package 与 source semantics。

## 规范结论

- `PluginManifest` 和 `LoadedPlugin` 应成为 `Tools` 域的稳定对象
- plugin package 路径、文件扩展名、安装介质可以变化
- 但 component declaration、enablement 与 source identity 语义必须稳定
