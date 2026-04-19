# Teammate Execution

## 职责

本文定义 teammate 体系：leader 如何管理长生命周期 worker，以及不同 backend 如何共享同一 teammate 语义。

## Teammate Identity

推荐最小对象：

```text
TeammateIdentity
  - agent_id
  - agent_name
  - team_name
  - color?
  - plan_mode_required
  - parent_session_id
```

约束：

- `agent_id` 必须稳定，供 routing / task lookup / UI projection 使用
- `agent_name + team_name` 必须足以定位 mailbox
- teammate identity 与 task id 不应混成一个对象

## Teammate Executor

推荐最小接口：

```text
TeammateExecutor
  - spawn(config) -> teammate_handle
  - send_message(agent_id, message)
  - terminate(agent_id, reason?)
  - kill(agent_id)
  - is_active(agent_id) -> boolean
```

## Backend Types

local teammate 默认至少允许以下 backend：

- `in-process`
- `tmux`
- `iterm2`

这些 backend 共享同一组 teammate 语义：

- 都是 leader 管理的 worker
- 都有稳定 teammate identity
- 都支持消息收发
- 都需要可观察 lifecycle

## In-Process 与 Pane-Backed 的差异

### in-process teammate

- 共享进程资源
- 依赖独立上下文隔离
- 可以直接更新本地 task state

### pane-backed teammate

- 不共享进程内状态
- 依赖外部 pane / session 作为承载
- 仍可共享同一 mailbox 语义

## 结果回流原则

teammate 的默认输出不会自动回灌给 leader。

默认模型应是：

- teammate 自己运行自己的 loop
- leader 通过 idle notification、mailbox 或 viewed transcript 了解状态
- 若 teammate 需要向 leader 汇报，应走显式消息通道

这与 delegated subagent 的“结果回到调用点”是两种不同模型。

## 规范结论

- teammate 是长生命周期 worker，不等于 subagent
- backend 可以不同，但 teammate identity、lifecycle 与 mailbox 语义必须一致
- teammate 输出默认不自动路由回 leader，必须通过显式消息或 viewed projection 暴露
