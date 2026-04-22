# Case: Command Surface Routing And Kind Boundary

## 目标

验证 command surface 作为 runtime 的 non-tool invocation surface 被稳定路由，并且 `local / local-jsx / prompt` 三类 command 的边界清晰。

当前语义锚点：

- `tools/command-surface/command-model.md`
- `tools/README.md`

## Preconditions

- runtime 支持 command surface 或语义等价的非 tool 调用入口
- system 至少支持两类不同 kind 的 command
- command invocation 可以被稳定路由到对应执行面

## Ingress

1. 准备一个 `local` command、一个 `local-jsx` command 和一个 `prompt` command，或语义等价对象
2. 通过 command surface 逐个触发它们
3. 观察 routing、执行面和结果对象
4. 验证 prompt command 可进一步进入 prompt-capability 路径，而不是被当成 tool call

## Expected Runtime Semantics

- `local`、`local-jsx`、`prompt` 是 command kind，而不是 tool kind
- command surface 属于 runtime 的 non-tool invocation surface
- `prompt` command 不因可被模型调用就退化成普通 tool
- routing 至少能区分 shell-like local execution、UI-like local rendering、prompt-capability activation 三条路径
- command 与 tool 必须保持对象层边界

## Expected Persistent Effects

- command invocation 至少留下可追溯的 runtime-visible result 或语义等价记录
- prompt command 触发后的后续上下文变化可被调用方稳定观察

## Allowed Variance

- `local-jsx` 可以表现为 UI action、rich local renderer 或语义等价 host surface
- command 入口可以是 slash、menu、programmatic invoke 或语义等价路由
- `prompt` command 可以被后续 skill-like activation 复用，但对象层仍是 command

## Failure Conditions

- command 与 tool 被建模成同一对象层且无法区分
- `prompt` command 被错误当成普通 tool call 处理
- runtime 无法按 command kind 稳定路由到不同执行面
- `local`、`local-jsx`、`prompt` 只是显示标签，不影响实际语义
