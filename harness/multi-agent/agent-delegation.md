# Agent Delegation

## 职责

本文定义 leader 如何委派 subagent，以及 delegated worker 的最小身份与回流语义。

## 核心语义

delegated subagent 是一种瞬时 worker。

它的特点是：

- 由 leader 显式发起
- 绑定明确的 agent definition 或 execution mode
- 可以同步完成，也可以异步进入 task 通道
- 默认不具备 teammate 那样的长期 team identity

## 推荐最小对象

```text
DelegatedAgentInvocation
  - agent_type
  - prompt
  - description
  - run_in_background?: boolean
  - isolation?
  - model?
  - parent_session_id?
  - invoking_request_id?
```

```text
DelegatedAgentIdentity
  - agent_id
  - agent_type
  - parent_session_id?
  - invoking_request_id?
  - invocation_kind: spawn | resume
```

## 执行模式

delegated subagent 至少应支持：

- synchronous delegation
  - 结果直接回到当前 turn
- background delegation
  - 结果通过 task / notification 回流
- remote delegation
  - worker 运行在远端执行面，但语义仍是 delegated worker

这些都是同一 delegated agent 模型的运行模式，不应被误建模成不同顶层能力。

## 与 Teammate 的边界

- subagent 默认围绕一次委派产生
- teammate 默认是长生命周期 worker
- subagent 的回流重点是“结果或通知”
- teammate 的回流重点是“显式消息、idle 状态、viewed transcript”

因此：

- teammate 不等于 subagent
- background agent 可以是 subagent 的一种运行模式
- 多 agent 规范不能只用 subagent 覆盖 teammate

## 规范结论

- delegated subagent 是 leader 显式发起的 worker
- delegated worker 必须具备稳定 identity，并能追溯到 parent session / request
- 同步回流与后台通知回流是同一委派模型的两种结果路径
