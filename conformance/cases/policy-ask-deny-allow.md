# Case: Policy Ask Deny Allow

## 目标

验证 `PolicyDecision` 的三类最终结果，以及它们与 `requires_action` / deny error 的映射关系。

## Preconditions

- 至少存在一个会直接 `allow` 的 tool
- 至少存在一个会 `ask` 的 tool
- 至少存在一个会 `deny` 或在当前 mode 下被安全拒绝的 tool
- runtime 支持结构化 `requires_action`

## Ingress

1. 触发一个可直接 allow 的工具调用
2. 触发一个需要 ask 的工具调用
3. 触发一个应被 deny 的工具调用
4. 若 ask 路径出现，注入 approval response

## Expected Runtime Semantics

- `allow`
  进入正常执行链路
- `ask`
  先产生 `PolicyDecision.ask`，再投影为结构化 `requires_action`
- `deny`
  不产生 resumable pending action，而是直接映射到 deny 终态或等价错误

## Expected Lifecycle

- allow 路径：`idle -> running -> idle`
- ask 路径：`idle -> running -> requires_action -> running -> idle`
- deny 路径：`idle -> running -> idle` 或语义等价的被拒绝终态

## Failure Conditions

- `ask` 被直接伪装成 deny
- `deny` 形成可恢复 pending action
- approval 恢复后没有绑定回原 `tool_use_id`
- background / async session 生成用户无法处理的悬空审批
