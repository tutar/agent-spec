# Case: Sandbox Deny

## 目标

验证 sandbox 对受限执行的拒绝语义，以及拒绝结果如何投影到 runtime。

## Preconditions

- runtime 启用了 `Execution Sandbox`、`Environment Sandbox` 或两者之一
- 至少存在一个会命中 sandbox deny policy 的操作
- session 初始状态为 `idle`

## Ingress

- 一条会触发受限操作的 user message

## Expected Lifecycle

- session lifecycle: `idle -> running -> idle` 或 `idle -> running -> requires_action -> idle`
- turn terminal state 必须能区分为：
  - sandbox denied
  - permission blocked
  - generic execution failure

## Expected Runtime Events

至少应出现以下之一：

- `tool_result`，其结果明确标记 sandbox deny
- `requires_action`，若实现把高风险操作先转成 approval
- `turn_failed`，但 failure category 必须保留 sandbox 语义

## Expected Semantics

- sandbox deny 不应被吞成普通 stdout/stderr 文本
- deny 必须保留结构化分类，供 gateway / UI / agent 使用
- deny 发生后，不应破坏 session 的后续可继续性

## Host Notes

- `Local` 通常更偏本地 execution sandbox deny，也可能来自 local environment sandbox
- `Cloud` 可能来自 remote environment policy 或 execution sandbox

两者都必须保留同一类外部拒绝语义。

## Failure Conditions

- deny 被降格为不可分类的普通 tool error
- deny 导致 session 无法继续后续 turn
- 不同 host 对同一 deny 给出不兼容的生命周期语义
