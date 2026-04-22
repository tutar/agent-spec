# Plugins

## 职责

`Plugins` 子模块定义 capability package、source loading、enablement 和 runtime delegation 的稳定规范。

对 `AgentRuntime` 来说，这一组接口回答的是：

- startup / refresh 时如何发现并加载 plugin sources
- plugin package 如何表达 commands、skills、hooks、MCP servers、agents、settings 等 component set
- loaded plugin 如何委托给既有子系统，而不是自行执行

## 阅读结构

- [plugin-package-and-manifest.md](plugin-package-and-manifest.md)
- [plugin-discovery-and-loading.md](plugin-discovery-and-loading.md)
- [plugin-runtime-delegation.md](plugin-runtime-delegation.md)

## 规范结论

- `Plugin` 不是 `Tool`
- `Plugin` 不是 `Command`
- `Plugin` 不是 `Skill`
- `Plugin` 不是 `MCP`
- `Plugin` 是 capability package / source loading / runtime delegation layer
