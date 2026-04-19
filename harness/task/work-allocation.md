# Work Allocation

## 目标

本文档定义本地多 worker 协作中的任务分配模型。

这里的 `task` 不是 runtime background task，而是团队协作中的待办项、分工项或工作单元。

它解决的是：

- 哪个 worker 负责什么
- 哪些工作项互相阻塞
- 团队如何认领、转交和释放工作
- 多 worker 如何共享一份可持久化的工作板

## 核心概念

### Work Item

`WorkItem` 是团队协作中的待办项。

它负责表达：

- 任务主题
- 任务描述
- 当前 owner
- 完成状态
- 阻塞关系

它不负责：

- 代表后台执行对象
- 承载 shell / agent / transcript branch 输出
- 替代 task lifecycle

### Work Status

推荐至少支持：

- `pending`
- `in_progress`
- `completed`

### Ownership

协作任务分配层应支持：

- 未认领
- 已认领
- 重新分配
- 释放归队

### Blocking Graph

协作任务层应允许显式表示：

- `blocks`
- `blocked_by`

这样 harness 才能支持依赖驱动的多 worker 协作。

## 推荐最小对象

```text
WorkItem
  - id: string
  - subject: string
  - description: string
  - owner?: string
  - status: pending | in_progress | completed
  - blocks: string[]
  - blocked_by: string[]
  - metadata?: map
```

```text
WorkBoard
  - board_id: string
  - items: WorkItem[]
```

## 推荐最小接口

```text
WorkAllocator
  - create_item(board_id, item) -> work_item
  - update_item(board_id, item_id, patch) -> work_item
  - list_items(board_id) -> work_items
  - claim_item(board_id, item_id, agent_id) -> result
  - release_item(board_id, item_id) -> result
  - resolve_item(board_id, item_id) -> result
  - get_agent_load(board_id) -> agent_statuses
```

推荐补充接口：

```text
DependencyManager
  - add_block(board_id, item_id, blocked_by_id)
  - remove_block(board_id, item_id, blocked_by_id)
  - get_ready_items(board_id) -> work_items
```

## 与 TaskManager 的边界

- `TaskManager`
  管运行时执行对象
- `WorkAllocator`
  管协作任务分配

允许但不要求两者建立映射：

- 一个 `WorkItem` 可以触发一个 `Task`
- 一个 `Task` 完成后可以更新某个 `WorkItem`
- 但两者不应被建模成同一个对象

## 与本地多 worker 协作的关系

在本地多 worker 系统中：

- harness runtime 决定 worker 如何被启动和跟踪
- work allocation 决定 worker 被分配什么工作

因此：

- 没有 work allocation，仍然可以有多 worker
- 但会缺少明确的协作分工和依赖管理

## 规范结论

- 多 worker 协作建议显式引入 work allocation 层
- work allocation 与 task 必须分开建模
- 只有把“执行”与“分工”拆开，many-agent 系统才不会把 task 概念混成一层
