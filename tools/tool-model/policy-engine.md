# Policy Engine

## 职责

`PolicyEngine` 负责在工具真正执行前做策略判定，并在需要时把会话推进到 `requires_action`。

权限不应被实现为简单弹窗，而应是一个可组合的策略系统。

## 必须覆盖的判定层

- 工具级 allow/deny
- 模式级限制
- 路径/工作目录限制
- sandbox 升级
- hook/classifier 审查
- async/background 会话降级策略

## 标准结果

```text
PolicyDecision
  - decision: allow | deny | ask | passthrough
  - reason
  - explanation?
  - requires_action_ref?
  - policy_source?
  - audit_metadata?
```

其中：

- `passthrough`
  仅用于策略链内部，表示当前检查器不做最终裁决，继续交给后续检查器
- 对外最终结果不应直接暴露 `passthrough`

canonical 对齐要求：

- `ask`
  最终应投影为 canonical `RequiresAction`
- `deny`
  最终应投影为 canonical `SdkError(permission_denied)` 或语义等价对象
- `allow`
  不应携带伪阻塞态

## `requires_action` 规范

阻塞不是异常字符串，而应是结构化对象。

```text
RequiresAction
  - action_type
  - tool_name
  - action_description
  - tool_use_id
  - request_id
  - input
  - resumable
  - details?
```

canonical 字段表见 [../object-model.md](../object-model.md)。

## 设计要求

- 可见工具过滤和执行时授权必须是两层逻辑
- 权限原因必须可解释
- 审批请求必须可序列化
- 非交互会话必须支持自动拒绝或安全降级
- 即使底层模型原生支持某些敏感能力，权限裁决仍应由 SDK 控制
- `ask` 与 `requires_action` 必须分层：前者是策略决策，后者是 session 阻塞态投影
- 拒绝过多时应允许从自动拒绝退化到 prompting
- 工具中断语义不应混进权限结果，而应由执行层单独建模
- policy 决策必须可审计、可序列化、可重放到 session 恢复链路中

## 默认策略顺序

1. 静态 deny rule
2. tool 输入校验
3. tool-specific permission check
4. hook/classifier
5. sandbox/working directory 检查
6. ask/deny/allow 决策

推荐补充：

7. denial tracking / fallback-to-prompting

无论内部顺序如何实现，下列边界必须稳定：

- visible tool filtering
  不是最终 execute-time decision
- input validation failure
  不能被误报为用户拒绝
- sandbox escalation / working directory restrictions
  可以影响 policy decision，但不应绕开审计语义

## 拒绝追踪

当策略系统包含 classifier / hook / 自动拒绝时，推荐引入 denial tracking。

最小要求：

```text
DenialTrackingState
  - consecutive_denials
  - total_denials
```

```text
DenialFallbackPolicy
  - record_denial(state) -> state
  - record_success(state) -> state
  - should_fallback_to_prompting(state) -> boolean
```

其职责是：

- 防止系统无限自动拒绝
- 在连续拒绝过多时退回到 prompting / ask 语义

若实现支持 async / background session：

- denial tracking 必须能在无前台 UI 时做安全降级
- fallback-to-prompting 只有在宿主可达用户回路时才应触发

## 与中断语义的边界

权限决策只回答：

- 能否执行
- 是否需要审批

它不回答：

- 执行后能否被取消
- 用户新输入到来时是 cancel 还是 block

这些应由 tool execution 层通过 `interruptBehavior` 或等价语义单独建模。

## Ask / RequiresAction / Resume 链路

规范上必须显式区分三层：

- `PolicyDecision.ask`
  表示当前策略要求外部动作
- `RequiresAction`
  表示 session/runtime 被结构化阻塞
- `approval / user input response`
  表示恢复该阻塞的外部输入

恢复约束：

- `ask` 必须绑定到原 `tool_use_id` 或语义等价 action ref
- `deny` 不应形成可恢复 pending action
- `ask` 转成 `requires_action` 后，session resume 必须能恢复原阻塞上下文

## Background / Async Session Semantics

对无交互前台的 session，规范至少要求：

- 默认 auto-deny，或进入安全降级路径
- 不得悬空生成用户永远无法处理的 approval
- 若宿主支持 delayed approval channel，必须显式建模该通道，而不是隐含假设 UI 存在

## 当前仓库映射

- 权限主逻辑见 [utils/permissions/permissions.ts](../../cc/utils/permissions/permissions.ts)
- 拒绝追踪见 [utils/permissions/denialTracking.ts](../../cc/utils/permissions/denialTracking.ts)
- 会话阻塞态见 [utils/sessionState.ts](../../cc/utils/sessionState.ts)
- 工具中断语义见 [Tool.ts](../../cc/Tool.ts) 和 [services/tools/StreamingToolExecutor.ts](../../cc/services/tools/StreamingToolExecutor.ts)

## 规范结论

- 权限系统必须结构化
- `passthrough` 可作为内部策略链语义，但不应成为最终对外结果
- `requires_action` 必须是一等 runtime 状态
- `ask` 与 `requires_action` 不能混成一个概念
- denial tracking 应被视为策略退化机制
- 所有策略判定都必须可追踪、可解释
- 权限责任不能下放给底层模型
- policy 产生的 `requires_action` 和 `permission_denied` 应分别映射到 canonical `RequiresAction` 与 `SdkError`
- `PolicyDecision` 应成为 `Tools` 模块的稳定共享对象之一
