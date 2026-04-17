# Work Allocation

## 目标

本文档定义多 agent 协作中的任务分配模型。

这里的 `task` 不是 runtime background task，而是团队协作中的待办项、分工项或工作单元。

它解决的是：

- 哪个 agent 负责什么
- 哪些工作项互相阻塞
- 团队如何认领、转交和释放工作
- 多 agent 如何共享一份可持久化的工作板

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
- 承载 shell / agent / remote session 输出
- 替代 runtime task lifecycle

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

这样 orchestration 才能支持依赖驱动的多 agent 协作。

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

- 一个 `WorkItem` 可以触发一个 `RuntimeTask`
- 一个 `RuntimeTask` 完成后可以更新某个 `WorkItem`
- 但两者不应被建模成同一个对象

## 与 Multi-Agent Orchestration 的关系

在多 agent 系统中：

- orchestration 决定 agent 如何被编排和寻址
- work allocation 决定 agent 被分配什么工作

因此：

- 没有 work allocation，仍然可以有多 agent
- 但会缺少明确的协作分工和依赖管理

## 默认实现映射

本仓库当前默认实现映射到：


默认实现特征：

- 任务列表按 `taskListId` 落盘
- 支持团队共享 task list
- 支持 owner、status、blocks、blockedBy
- 支持 claim / release / reset / load 检查
- 与 teammate / team name 解析打通

## 规范结论

- 多 agent 协作建议显式引入 work allocation 层
- work allocation 与 runtime task 必须分开建模
- 只有把“执行”与“分工”拆开，many-agent 系统才不会把 task 概念混成一层
