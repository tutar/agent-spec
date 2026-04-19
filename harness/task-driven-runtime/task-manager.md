# Task Manager

## 职责

`TaskManager` 负责管理 `Harness` 域内的长生命周期执行对象。

这里的 `task` 指 runtime background task，而不是团队协作里的待办项或任务板条目。

## 解决的问题

- 如何统一管理后台 shell、agent、workflow、monitor 等异步执行对象
- 如何为这些对象提供统一的状态机、输出句柄、通知和终止语义
- 如何让 UI、agent、session restore 不必理解每种执行对象的内部细节
- 如何把 agent loop、shell process、transcript branch 包装成同一层运行时对象

## 核心模型

`RuntimeTask` 是一个被 harness 托管的执行单元。

它至少应具有：

- 稳定 identity
- 明确 task type
- 生命周期状态
- 输出句柄
- 可通知性
- 可终止性

它不等于：

- tool call
- transcript message
- 团队任务列表中的待办项
- cloud control plane 对象

## 推荐标准对象

```text
RuntimeTask
  - id: string
  - type: string
  - description: string
  - status: pending | running | completed | failed | killed
  - start_time: timestamp
  - end_time?: timestamp
  - tool_use_id?: string
  - output_ref?: output_handle
  - notified: boolean
```

```text
TaskOutputHandle
  - output_ref: string
  - transport: file | stream | remote_log | transcript_branch
  - cursor?: string | number
```

## 推荐最小接口

```text
TaskManager
  - spawn(task_spec) -> task_handle
  - append_event(task_handle, task_event)
  - attach_output(task_handle, output_handle)
  - kill(task_handle)
  - list(selector?) -> runtime_tasks
  - get(task_handle) -> runtime_task
  - read_events(task_handle, cursor?) -> task_event_slice
  - await(task_handle) -> task_terminal_state
```

如果宿主希望暴露更直接的过程式操作，
例如 `complete()`、`fail()`、`update()`，
也应把它们理解为 `append_event()` 的 convenience facade，
而不是另一套独立状态机。

## 与 Agent 的关系

agent 可以运行在 task 之上，但 agent 不等于 task。

推荐关系是：

- `agent loop`
  负责推理、tool use、turn continuation
- `runtime task`
  负责后台生命周期、输出、通知、恢复

## 与协作任务列表的边界

本规范中的 `TaskManager` 不等于团队协作中的 task list。

- `TaskManager`
  解决“谁在运行、输出在哪、何时结束”
- `WorkAllocator`
  解决“谁负责哪个待办、哪些待办互相阻塞”

团队协作任务列表见 [work-allocation.md](work-allocation.md)。

## 规范结论

- runtime task 必须是一等 harness 对象
- task manager 管的是执行生命周期，不是业务待办
- 后台任务必须支持状态同步、输出句柄、通知和 kill
- task 语义必须独立于具体语言线程模型或协程模型
- `task_handle + task_event + output_handle` 应优先于进程内 task 对象引用
- direct-call task API 如存在，也必须严格由 task event 语义推导
- task terminal state 与 retryability 语义应与 [../runtime-core/failure-and-terminal-states.md](../runtime-core/failure-and-terminal-states.md) 对齐
