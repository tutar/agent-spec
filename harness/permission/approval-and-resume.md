# Approval And Resume

## 职责

本文定义 permission-triggered approval 的运行时语义：

- `ask -> requires_action` 的投影
- approval channel
- delayed approval
- worker / leader / bridge / channel 审批路由
- resume 后如何绑定回原执行点

## 基本链路

规范上必须显式区分三层：

- `PolicyDecision.ask`
  当前策略需要外部动作
- `RequiresAction(approval)`
  runtime 被结构化阻塞
- approval response
  恢复该阻塞的外部输入

最小要求：

- `ask` 必须能绑定回原 tool action 或等价 action ref
- approval response 必须能恢复到原阻塞点
- `deny` 不得形成 resumable pending approval

## Approval Channel

推荐最小对象：

```text
ApprovalChannel
  - channel_type: local_ui | leader_bridge | remote_callback | delayed_queue
  - request_id
  - target_session_id
  - target_agent_id?
```

实现可以有不同 transport，但必须显式建模 approval route。

## 场景语义

### Interactive Main Agent

- `ask` 投影为本地可处理的 `RequiresAction`
- approval 由当前用户回路恢复

### Background Or Headless Worker

- 若存在 automated checks，应先运行 automated checks
- 若存在可达 approval channel，可桥接到上层 reviewer / leader
- 若不存在可达 approval channel，必须 auto-deny

### Worker To Leader Approval

- worker 不应假定自己总能直接展示 approval UI
- worker 可以把 approval request 路由到 leader
- leader 的 approve / deny 必须绑定回原 worker action

### Remote / Channel Approval

- 宿主若支持 bridge、channel callback 或远端控制面审批，应显式表示该通道
- 远端审批的 transport 可以变化，但阻塞与恢复语义不能变化

## 与 Session 的边界

- `Session`
  负责 `RequiresAction` 的 durable 恢复语义
- `Approval And Resume`
  负责 permission approval 的绑定、路由和恢复链

恢复要求：

- restore 后若 approval 仍 pending，session 必须重建同一阻塞上下文
- approval 处理后，系统必须继续原 action，而不是重新构造一个无关 tool call

## Failure Conditions

以下都应视为不合规：

- `ask` 直接被伪装成普通 error
- background session 生成用户不可达 approval
- approval response 无法映射回原 tool action
- deny 被错误建模成可恢复 pending action

## 规范结论

- permission approval 是 runtime block，不是 UI 文案
- approval channel 必须显式建模
- background / worker / remote 场景必须有清晰的降级或桥接语义
- restore 后必须能重建原 permission block，而不是只恢复描述文本
