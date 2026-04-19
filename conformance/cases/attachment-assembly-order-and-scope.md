# Case: Attachment Assembly Order And Scope

## 目标

验证 attachment 作为独立 context plane 时，具有稳定顺序与明确 scope，而不是无序文件列表。

## Preconditions

- runtime 支持 attachment assembly
- 至少存在以下 attachment 类别中的三类：
  - `user_input`
  - `all_thread`
  - `main_thread_only`
  - `agent_scoped`

## Ingress

1. 发起一轮带显式用户附件的 turn
2. 在同一轮中添加一个全线程可见 attachment
3. 添加一个只对主线程可见的 attachment
4. 若 runtime 支持 worker/subagent，再添加一个 `agent_scoped` attachment

## Expected Runtime Semantics

- attachment 不得被视为无序集合
- attachment 至少应支持稳定顺序：
  - `user_input`
  - `all_thread`
  - `main_thread_only`
  - `agent_scoped`
- 不同 scope 的 attachment 可见性必须 deterministic
- attachment 与 transcript 必须语义分离
- 大 payload 可通过 `payload_ref` 或语义等价引用暴露，而不是强制全量内联

## Expected Runtime Effects

- 主线程与 worker/subagent 对同一轮 attachment 的可见集合可以不同，但必须符合 scope 规则
- 若 runtime 暴露 attachment envelope 或等价调试信息，应能观察到稳定 ordering 和 scope

## Allowed Variance

- 若实现没有 worker/subagent，则 `agent_scoped` 可用单线程局部 scope 的语义等价场景替代
- attachment payload 可使用 inline、file ref、object ref 或其它稳定引用

## Failure Conditions

- attachment 顺序不稳定
- `main_thread_only` attachment 泄露给全部 worker
- `agent_scoped` attachment 被广播到所有线程
- attachment 被强制并入 transcript 本体且失去 scope
