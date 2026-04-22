# Case: Builtin Tool Baseline And Gating

## 目标

验证 runtime 具有 built-in capability baseline，并且 mode / feature gating 调整的是暴露面，而不是改写 `Tools` 对象模型。

当前语义锚点：

- `tools/builtin/builtin-tool-baseline.md`
- `tools/tool-model/tool-registry.md`

## Preconditions

- runtime 支持 built-in tool families 或语义等价的默认 capability surface
- host 支持至少一种 capability gating 机制，例如 mode、feature flag 或 profile gating
- registry 或语义等价对象可查询当前可用 capability surface

## Ingress

1. 在默认模式下读取 built-in capability surface
2. 切换到一个更受限或更精简的 gating 模式
3. 再次读取 capability surface
4. 对比前后两次 surface 的 family、identity 与对象边界

## Expected Runtime Semantics

- 默认 runtime 至少暴露一组 built-in capability family
- gating 调整的是 capability exposure，而不是重定义 tool object model
- built-in baseline 是 runtime 默认 capability surface，不是新的顶层模块
- gating 后 capability 可以减少、隐藏或降级，但已暴露对象仍保持同一对象模型

## Expected Persistent Effects

- capability surface 变化对调用方是可观察的
- gating 切换后 registry 或语义等价 surface 可反映新的可用集

## Allowed Variance

- built-in capability family 的具体成员可以不同
- gating 可以是 static config、startup mode 或动态 profile 切换

## Failure Conditions

- runtime 默认没有任何 built-in capability baseline
- gating 通过改写对象模型而不是调整 exposure 生效
- 同一 capability 在不同 mode 下 identity 漂移为不同对象种类
- built-in baseline 被错误建模成顶层独立模块
