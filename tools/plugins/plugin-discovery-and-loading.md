# Plugin Discovery And Loading

## 职责

本文件定义 plugin source 是如何被发现、验证、加载、合并并进入当前 runtime 的。

对 `AgentRuntime` 来说，这一页定义的是 startup / refresh 时调用的 plugin discovery and loading surface。

## 稳定接口

推荐最小接口：

```text
PluginLoader
  - discover_plugins(scope) -> discovered_plugin_refs
  - load_plugins(runtime_context) -> PluginLoadResult
  - merge_plugin_sources(sources) -> loaded_plugins
  - cache_plugin_settings(loaded_plugins)
```

推荐对象：

```text
PluginSource
  - kind: builtin | installed | inline
  - source_id
  - precedence
  - repository_ref?
  - managed_policy?
```

```text
PluginLoadResult
  - loaded_plugins[]
  - disabled_plugins[]
  - errors[]
  - merged_settings?
```

## 来源模型

runtime 至少应允许三类 source family：

- `builtin`
  - shipped with the host/runtime
- `installed`
  - marketplace or repository-installed plugin
- `inline`
  - session-only override or explicit plugin-dir injection

## Source Precedence

推荐默认顺序：

1. inline
2. installed
3. builtin

约束：

- inline source 可以覆盖 installed source 的同名 plugin
- managed policy 可以阻止本地 override
- builtin source 保留为独立 source family，不与 installed 混写

## 发现与加载语义

`AgentRuntime` 在 startup / refresh 时应至少完成：

1. discover configured sources
2. validate manifests and component declarations
3. determine enablement state
4. load builtin / installed / inline sources
5. merge sources by precedence
6. materialize `LoadedPlugin[]`
7. merge plugin-contributed settings

实现可以区分：

- cache-only load
- full load

但外部稳定语义必须保持：

- same source precedence
- same enablement semantics
- same delegated component set

## 默认实现映射

本地实现可以把发现根目录放在：

- `.openagent/plugins`

并在 session-only override 中支持显式 plugin-dir 注入。

这些仍然只是默认实现映射，不是跨语言强约束。

## 规范结论

- plugin loading 是 runtime startup / refresh capability assembly 的正式阶段
- source precedence、enablement 和 merged settings 必须 deterministic
- plugin loading 失败不应抹掉其它 source 的稳定结果
