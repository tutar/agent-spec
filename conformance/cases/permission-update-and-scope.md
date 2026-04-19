# Case: Permission Update And Scope

## 目标

验证 permission update 的作用域、持久化边界和只读 source 约束。

## Preconditions

- runtime 支持结构化 `PermissionUpdate`
- 至少存在一个 session-scoped destination
- 至少存在一个 persisted destination
- 若实现支持 managed-only source，还应能区分可写与只读来源

## Ingress

1. 提交一个 session-scoped rule update
2. 在同一 session 中验证该规则已生效
3. 启动新 session，验证该规则未自动持久化
4. 提交一个 persisted rule update，并验证其在后续会话中仍生效
5. 尝试对只读 source 提交 update
6. 若实现支持 managed-only mode，启用后验证非受管规则不再生效或不再可写

## Expected Runtime Semantics

- session update 只影响当前会话
- persisted update 应在后续会话中继续生效
- 只读 source 不得被误当成可写 destination
- managed-only mode 下，受管来源之外的持久化入口应被隐藏、拒绝或忽略

## Allowed Variance

- persisted destination 的具体存储介质可不同
- managed-only mode 的 UI 表现可不同

## Failure Conditions

- session update 泄漏到后续会话
- persisted update 只在内存中生效
- 只读 source 被错误修改
- managed-only mode 下非受管规则仍继续生效且无解释
