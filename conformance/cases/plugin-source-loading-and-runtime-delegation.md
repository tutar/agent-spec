# Case: Plugin Source Loading And Runtime Delegation

## 目标

验证 runtime 在 startup / refresh 时的 plugin source loading、source precedence 和 runtime delegation 语义。

当前语义锚点：

- `tools/plugins/plugin-discovery-and-loading.md`
- `tools/plugins/plugin-runtime-delegation.md`

## Preconditions

- runtime 支持 plugin discovery / loading
- 至少存在 builtin、installed 或 inline 三类 source 中的两类

## Ingress

1. 触发一次 startup-time plugin loading
2. 若实现支持 refresh，再触发一次 refresh
3. 若存在同名 plugin，检查 source merge 结果

## Expected Runtime Semantics

- runtime 在 startup / refresh 时调用 plugin loading surface
- source precedence deterministic
- enablement state deterministic
- source merge 不抹掉 builtin / installed / inline family distinction

## Expected Delegation Behavior

- loaded plugin 的 component set 被稳定委托给 commands / skills / hooks / MCP / agents / settings 等既有子系统
- settings contribution 不应改变 plugin 的 package 角色
- plugin loading 失败不应破坏其它 source 的稳定结果

## Allowed Variance

- 不同实现可使用不同的 cache、repository、marketplace、inline source 载体
- source 路径、安装目录、manifest 文件名可不同

## Failure Conditions

- startup / refresh 不经过统一 plugin loading surface
- source precedence 不稳定
- inline / installed / builtin source family 被错误抹平
- plugin 组件未通过 delegation 进入既有子系统
