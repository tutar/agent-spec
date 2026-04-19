# Background Agent

## 职责

`BackgroundAgent` 定义 harness 管理的后台 agent 执行单元。

它负责：

- 脱离当前前台 turn 持续运行
- 挂接到 `TaskManager` 的统一 lifecycle
- 把 progress、notification、terminal state 投影回 session / gateway

它不负责：

- cloud control plane wake / reprovision
- tool schema 定义
- transcript durable storage

## 推荐最小对象

```text
BackgroundAgentHandle
  - agent_id
  - task_id
  - session_id?
  - status
  - output_ref?
```

## 推荐最小接口

```text
BackgroundAgent
  - spawn(request) -> background_agent_handle
  - send_input(handle, input)
  - await(handle) -> terminal_state
  - terminate(handle) -> result
```

## 规范结论

- background agent 是 harness 管理的本地运行时对象，不是 cloud orchestration 对象
- background agent 应显式映射到 `TaskManager` 的 task lifecycle
- progress、notification 和 terminal state 应可被外部观察与恢复
