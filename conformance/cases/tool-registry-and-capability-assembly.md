# Case: Tool Registry And Capability Assembly

## 目标

验证 `ToolRegistry` 或语义等价对象是 runtime capability assembly 的结果，并能统一承载 built-in、protocol-adapted 与 package-contributed tool surface。

当前语义锚点：

- `tools/tool-model/tool-registry.md`
- `tools/plugins/plugin-runtime-delegation.md`
- `tools/mcp/runtime-adaptation.md`

## Preconditions

- runtime 支持 registry 或语义等价 capability assembly surface
- 至少存在两类不同来源的 tool capability
- runtime 可查询 assembled tool surface

## Ingress

1. 启动 runtime 并完成 capability assembly
2. 读取 assembled tool surface
3. 引入额外 capability source，例如 protocol-adapted tool 或 plugin-contributed tool
4. 刷新 capability assembly 并再次读取 surface

## Expected Runtime Semantics

- registry 是 runtime capability assembly 的结果，不是单一 source 的私有清单
- built-in、protocol-adapted 与 package-contributed tool 可进入同一 assembled surface
- source family 可区分，但最终 tool object model 一致
- capability assembly 不得要求调用方分别查询每个 source family 才能得知可用 tools

## Expected Persistent Effects

- refresh 后新的 assembled surface 对 runtime 调用方可见
- 已有 capability identity 在 source 未变化时保持稳定

## Allowed Variance

- registry 可以是 map、catalog、index 或语义等价对象
- source merge 可以发生在 startup、refresh 或 lazy capability discovery 时

## Failure Conditions

- registry 只暴露 built-in tools，无法承载其他 source family
- 不同 source family 进入 registry 后对象模型不一致
- capability assembly 结果只存在于实现内部，调用方无法稳定查询
- source refresh 导致无关 capability identity 全量漂移
