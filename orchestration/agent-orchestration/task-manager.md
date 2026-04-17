# Task Manager

## 职责

`TaskManager` 负责管理运行时长生命周期执行对象。

这类对象可能由 tool 触发，但不应被简化成普通 tool result。

这里的 `task` 指的是 runtime background task，而不是团队协作里的待办项或任务板条目。

## 解决的问题

- 如何统一管理后台 shell、agent、remote agent、workflow、monitor 等异步执行对象
- 如何为这些对象提供统一的状态机、输出句柄、通知和终止语义
- 如何让 UI、agent、session restore 不必理解每种执行对象的内部细节
- 如何把 agent loop、shell process、remote session 包装成同一层控制面对象

## 核心模型

### Runtime Task

`RuntimeTask` 是一个被 orchestration 托管的执行单元。

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
- sandbox 本身

### TaskStatus

推荐至少支持：

- `pending`
- `running`
- `completed`
- `failed`
- `killed`

其中：

- `completed / failed / killed`
  都是 terminal state

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

## 适合进入 TaskManager 的对象

- 后台 shell 命令
- 子 agent
- remote agent
- workflow
- monitor
- swarm teammate

## 为什么 task 要独立于 tool

tool 是模型侧调用接口，task 是运行时侧执行对象。二者混在一起会导致：

- 无法统一管理后台生命周期
- 无法在 UI 中独立展示异步工作
- 无法做 kill、resume、notification
- 无法表达 agent/subagent 这类长期对象

## 与 Agent 的关系

agent 可以运行在 task 之上，但 agent 不等于 task。

推荐关系是：

- `agent loop`
  负责推理、tool use、turn continuation
- `runtime task`
  负责 agent 的后台生命周期、输出、通知、恢复

因此 task manager 应至少能承载：

- 同步 worker 的内部跟踪
- 后台 agent 的公开 task lifecycle
- persistent teammate 的长存活状态
- remote agent 的本地 task handle

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

推荐补充接口：

```text
TaskEvent
  - task_id
  - type: started | progress | notification | completed | failed | killed
  - timestamp
  - payload
```

```text
TaskOutputStore
  - append(output_ref, chunk)
  - read_delta(output_ref, cursor) -> delta
  - evict(output_ref)
```

如果本地宿主希望暴露更直接的过程式操作，
例如 `complete()`、`fail()`、`update()`，
也应把它们理解为 `append_event()` 的 convenience facade，
而不是另一套独立状态机。

## 默认实现映射


典型默认实现：

- `local_bash`
- `local_agent`
- `remote_agent`
- `in_process_teammate`
- `dream`

默认实现特征：

- task 统一带有 `outputFile / outputOffset / notified`
- task 注册、更新、通知、回收走统一框架
- task output 与 task lifecycle 解耦，使用独立 output handle
- agent、shell、remote session 都被包装成 runtime task

## 与协作任务列表的边界

本规范中的 `TaskManager` 不等于团队协作中的 task list。

- `TaskManager`
  解决“谁在运行、输出在哪、何时结束”
- team task list
  解决“谁负责哪个待办、哪些待办互相阻塞”

团队协作任务列表见 [work-allocation.md](work-allocation.md)。

## 规范结论

- runtime task 必须是一等 orchestration 对象
- task manager 管的是执行生命周期，不是业务待办
- 后台任务必须支持状态同步、输出句柄、通知和 kill
- task 语义必须独立于具体语言线程模型或协程模型
- task manager 应足以支撑 shell、agent、remote、teammate 等多类执行对象
- `task_handle + task_event + output_handle` 应优先于进程内 task 对象引用
- direct-call task API 如存在，也必须严格由 task event 语义推导
- task terminal state 与 retryability 语义应与 [../../harness/runtime-core/failure-and-terminal-states.md](../../harness/runtime-core/failure-and-terminal-states.md) 对齐
