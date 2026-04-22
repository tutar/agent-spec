# Case: Plugin Package Vs Runtime Capability Boundary

## 目标

验证 plugin 在 `Tools` 域中的角色边界。

当前语义锚点：

- `tools/plugins/README.md`
- `tools/plugins/plugin-package-and-manifest.md`
- `tools/plugins/plugin-runtime-delegation.md`

## Preconditions

- runtime 支持 plugin package / source loading
- 至少存在一个 plugin，且该 plugin 提供不止一种 component

## Ingress

1. 发现并加载一个 plugin package
2. 检查其 manifest / loaded plugin 结构
3. 检查其 components 被委托到哪些既有子系统

## Expected Runtime Semantics

- plugin 被建模成 package / source / delegation layer
- plugin 不被错误建模成 `Tool`
- plugin 不被错误建模成 `Skill`
- plugin 不被错误建模成 `Command`
- plugin 不被错误建模成 `MCP`

## Expected Delegation Behavior

- command-like components 进入 command surface
- skill components 进入 skill registry / bridge
- MCP server components 进入 MCP 子系统
- hooks / agents / settings components 进入各自既有子系统

## Failure Conditions

- plugin 被实现成统一执行引擎
- plugin 直接承担 tool execution 语义
- plugin 抹掉 component provenance
- plugin package 与 capability object 边界混淆
