# Case: Startup Context Lifecycle

## 目标

验证 startup-only context 与 per-turn context 的生命周期分层稳定，且不会污染 bootstrap prompt 或稳定 agent prompt layer。

## Preconditions

- runtime 支持至少以下 lifecycle context 中的三类：
  - `session_start`
  - `agent_start`
  - `turn_zero`
  - `resume_start`
- session 支持 resume 或语义等价的 reentry

## Ingress

1. 启动主 session，并注入 `session_start`
2. 触发首轮 turn，注入一次性 `turn_zero` briefing
3. 若 runtime 支持 delegated worker，启动一个 worker 并注入 `agent_start`
4. 触发一次 resume / reentry，并注入 `resume_start`

## Expected Runtime Semantics

- `session_start`、`agent_start`、`turn_zero`、`resume_start` 必须可区分
- startup-only context 不得被误建模为 bootstrap prompt 本体
- `agent_start` 不得污染主 session 的 `session_start`
- `turn_zero` 只应影响首轮或显式 reentry 策略指定的轮次
- `resume_start` 必须允许重新宣布恢复所需上下文
- 稳定 agent prompt layer 不得因为 startup-only context 被永久改写

## Expected Persistent Effects

- startup-only context 的 transcript 可见性和 dedup 语义必须 deterministic
- resume 后不应因为丢失 startup lifecycle 语义而要求重建整个 session prompt

## Allowed Variance

- 若实现不支持 worker/subagent，可用单独 agent-local startup path 语义等价替代 `agent_start`
- 是否把 startup context 投影到 transcript，可由 host profile 决定，但必须稳定且可解释

## Failure Conditions

- `turn_zero` 变成每轮持续存在的稳定 prompt layer
- `agent_start` 泄露到主 session startup 语义
- `resume_start` 被忽略，导致恢复所需上下文无法重新宣布
- startup-only context 只能通过修改 bootstrap prompt 才能实现
