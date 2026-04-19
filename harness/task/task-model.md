# Task Model

## 职责

本文定义 `Harness` 域内 task 子系统的共享对象模型。

这里的 `task` 是被 harness 托管的长生命周期执行对象，用于统一表示：

- shell 执行
- local agent 执行
- remote agent 执行
- workflow / monitor / verifier 等后台执行单元

它不等于：

- `AgentRuntime` 的 turn state machine
- agent definition
- tool call
- transcript message
- 团队协作任务列表中的待办项

## 核心对象

### TaskStatus

推荐最小状态集合：

```text
TaskStatus
  - pending
  - running
  - completed
  - failed
  - killed
```

这些状态只描述 task 自己的执行状态。

它们不等于 `AgentRuntime` 的 `terminal state`，
也不应被解释为 turn 级 `requires_action`、`aborted`、`timed_out` 的别名。

### Task

推荐最小对象：

```text
Task
  - id: string
  - type: string
  - status: TaskStatus
  - description: string
  - start_time: timestamp
  - end_time?: timestamp
  - tool_use_id?: string
  - output_ref: output_handle
  - output_cursor?: string | number
  - notified: boolean
```

约束：

- `id` 必须稳定，供恢复、观察和控制使用
- `type` 表示执行类别，不等于 agent 身份标签
- `output_ref` 是 task 的稳定输出句柄
- `notified` 只表示 terminal notification 是否已稳定投影

### TaskOutputHandle

```text
TaskOutputHandle
  - output_ref: string
  - transport: file | stream | remote_log | transcript_branch
  - cursor?: string | number
```

约束：

- 具体 transport 可以变化
- `output_ref + cursor` 语义必须稳定
- 大输出不得要求全量内存驻留

### TaskNotificationState

```text
TaskNotificationState
  - notified: boolean
  - last_notification_cursor?: string | number
```

### TaskRetention

```text
TaskRetention
  - retained: boolean
  - evict_after?: timestamp
  - view_owner?: string
```

## Task Registry

task 的单一事实来源应是 `TaskRegistry`。

推荐最小对象：

```text
TaskRegistry
  - tasks: map<task_id, task>
```

语义要求：

- task 必须先注册到 `TaskRegistry`，再进入可观察生命周期
- 外部观察者应通过 registry 查询 task，而不是依赖进程内临时对象引用
- restore 后 task 的最近状态、输出句柄与可恢复元数据，应从 registry 或等价 durable state 重建

## Implementation Registry

除了 state registry，宿主还应支持按 `task.type` 分发的 implementation registry。

推荐最小接口：

```text
TaskImplementationRegistry
  - get(task_type) -> task_adapter
```

它用于：

- kill / terminate 分发
- task type 专属控制逻辑
- type-specific 输出或恢复适配

约束：

- implementation registry 不替代 `TaskRegistry`
- state registry 负责“现在有什么 task、状态如何”
- implementation registry 负责“某种 task type 如何被控制”

## 典型 task 类型

规范不强制具体命名，但应允许至少以下来源进入 task 体系：

- local shell task
- local agent task
- remote agent task
- teammate / worker task
- workflow / monitor / verifier task

因此：

- task 不等于 agent
- agent 只是 task 的一种来源
- shell、workflow、monitor 同样可以接入 task 模型

## 与其它规范的边界

- task 的生命周期见 [task-lifecycle.md](task-lifecycle.md)
- manager 的职责与接口见 [task-manager.md](task-manager.md)
- background agent 如何映射到 task，见 [background-agent.md](background-agent.md)
- 协作工作分配不属于 task 模型，见 [work-allocation.md](work-allocation.md)

## 规范结论

- task 是独立执行层对象，不属于 `AgentRuntime` 的 turn state
- `TaskStatus` 与 runtime terminal taxonomy 必须分开
- `TaskRegistry` 必须作为 task 的单一事实来源显式存在
- `TaskRegistry` 与按 type 分发的 implementation registry 应分层建模
