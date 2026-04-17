# Capability Surface

## 目标

本文件定义 agent 如何把不同来源的能力组织成统一的宿主可见能力面。

它要解决的问题不是某个具体 UI 页面，而是：

- 不同来源的能力如何共存
- 宿主如何把这些能力投影到统一入口
- built-in、bundled、plugin、mcp、user 等来源如何被区分

## 核心概念

### 1. Builtin Capability

随宿主或 agent 本体分发的能力。

典型包括：

- built-in tools
- built-in commands
- 宿主自带的固定 orchestration entry

特点：

- 无需用户安装
- 通常始终可用，或仅受 feature/profile/policy 过滤
- 构成 capability baseline

### 2. Bundled Capability

随宿主分发，但语义上属于可复用扩展内容的能力。

典型包括：

- bundled skills
- bundled workflows
- bundled extension assets

特点：

- 由宿主一起发布
- 不一定属于核心 runtime
- 可以与 built-in command 一起投影给用户，但内部来源不同

### 3. Installed Extension Capability

由插件、marketplace 包、workspace 内容或管理员配置引入的能力。

典型包括：

- plugin skills
- plugin commands
- plugin agents
- plugin MCP / LSP integration
- user/project skills

### 4. Remote Capability

由外部协议或远端系统动态接入的能力。

典型包括：

- MCP tools
- MCP prompts
- MCP resources
- remote hands / remote tool proxies

## 统一来源模型

所有能力对象都应保留来源信息。

推荐最小字段：

```text
CapabilityOrigin
  - origin_type: builtin | bundled | plugin | user | project | managed | mcp | remote
  - package_id?
  - provider_id?
  - installation_scope?
```

该来源模型应适用于：

- tool
- command
- skill
- plugin component

## Unified Command Surface

宿主可以提供统一的 command surface，将不同来源的 invocable capability 投影到同一入口。

在 Claude Code 形态中，这通常体现为 slash command / command palette / launcher。

这个统一入口可以混合展示：

- built-in commands
- bundled skills
- user/project skills
- plugin commands
- plugin skills
- workflow commands

但必须满足：

- 展示面统一，不代表内部对象语义相同
- `command`、`skill`、`tool` 的边界不能因为混合展示而消失
- 来源信息必须可追溯

## 稳定接口

推荐最小接口：

```text
CapabilitySurface
  - list_capabilities(scope, filters) -> capabilities[]
  - list_command_surface(scope, filters) -> invocable_entries[]
  - resolve_capability(id) -> capability
  - project_for_host(host_profile) -> host_capability_view
```

推荐最小对象：

```text
InvocableEntry
  - entry_id
  - entry_type: command | skill | workflow | action
  - display_name
  - description
  - source_origin
  - invocation_mode: user | model | both
```

## 规范要求

### 1. Builtin 与 Bundled 必须区分

虽然宿主可以把 built-in commands 与 bundled skills 混合展示，但规范上必须保留两者差别：

- built-in
  宿主核心能力
- bundled
  宿主附带分发的扩展能力

### 2. Unified Command Surface 是宿主投影，不是对象模型

统一命令面只是投影层。

它不应抹平：

- command
- skill
- tool
- plugin component

之间的对象边界。

### 3. Host profile 可以决定投影形态，但不能改变语义

- `Local`
  常投影为本地可调用入口
- `Cloud`
  常投影为 API-discoverable invocable actions

两者都可以提供统一入口，但不应改变 capability origin 语义。

### 4. Builtin Extension Package 是可选规范能力

宿主可以声明一组官方自带扩展包，使其：

- 随宿主发布
- 可启用/禁用
- 共享 plugin/extension 模型

但这不是五个核心模块之一，而是扩展分发层能力。

## 默认实现映射

当前仓库中的默认实现映射为：


从当前仓库可见：

- 命令面会混合展示 built-in commands 与 bundled skills
- built-in plugin registry 存在，但当前快照中尚无实际 built-in plugin 组件注册

## 规范结论

- `Capability Surface` 应作为共享规范存在
- 它应解释不同来源能力如何共存并投影给宿主
- built-in commands 与 bundled skills 可以混合展示，但来源语义必须保留
- built-in plugin 更适合作为 extension packaging 的可选能力，而不是新的核心模块
