# Case: Skill Invocation Mode And Context Boundary

## 目标

验证 skill invocation 的 `inline / fork` 模式分层，以及 `SkillTool` 与 skill 本体、turn context 之间的边界。

当前语义锚点：

- `tools/skills/skill-definition.md`
- `tools/skills/skill-context-management.md`
- `tools/skills/skill-disclosure-and-activation.md`

## Preconditions

- runtime 支持 skill capability 或语义等价 prompt-capability surface
- 至少支持两种 invocation mode，或支持语义等价的 inline/fork 分层
- skill activation 可影响当前 context 或派生出独立 execution context

## Ingress

1. 准备一个 `inline` skill 和一个 `fork` skill，或语义等价配置
2. 触发两类 skill activation
3. 观察当前 turn context、execution boundary 与 output/result handoff
4. 比较两类 mode 的 context 影响面

## Expected Runtime Semantics

- `SkillTool` 是 bridge surface，不等于 skill definition 本体
- `inline` mode 在当前 turn context 内生效
- `fork` mode 派生出独立 execution context、subagent 或语义等价隔离面
- skill activation 不得把 context protection 与 invocation mode 混成同一概念

## Expected Persistent Effects

- 两类 invocation mode 的结果都可被调用方稳定追溯
- fork mode 的独立 execution boundary 对外部可观察

## Allowed Variance

- fork 可以是 subagent、background execution unit 或语义等价派生执行面
- inline 可以是 prompt splice、context injection 或语义等价当前上下文扩展

## Failure Conditions

- inline 与 fork 只有命名差异，没有实际 boundary 差异
- `SkillTool` 被错误建模成 skill definition 本体
- skill activation 破坏当前 context protection 或无法区分当前/派生上下文
