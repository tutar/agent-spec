# Case: Work Allocation Vs Runtime Task Boundary

## 目标

验证协作 work allocation 与 runtime task lifecycle 分层建模，而不是把待办项、分工项与执行对象混成同一个对象。

当前语义锚点：

- `harness/task/work-allocation.md`
- `harness/task/task-model.md`
- `harness/task/task-manager.md`

## Preconditions

- host 支持多 worker 协作，或支持语义等价的 shared work board
- system 支持显式 work item owner、status 与 blocking graph
- runtime task 与 work item 至少可以建立可选映射，而不是强制同一 identity

## Ingress

1. 创建一个包含至少两个 work item 的 work board
2. 为其中一个 item 添加 `blocked_by` 或语义等价依赖
3. 认领一个 ready item，并触发与之相关的 runtime task
4. 观察 work item 与 runtime task 的状态推进
5. 完成或失败该 runtime task，并验证 work board 更新
6. 释放或重新分配另一个 work item，验证 owner 与 ready set 变化

## Expected Runtime Semantics

- `WorkItem` 与 `Task` 必须是两个不同层次的对象
- work item 至少可表达 `pending / in_progress / completed`
- work item 至少可表达 owner、`blocks` 与 `blocked_by`
- claim、release、reassign 不得改变 runtime task identity
- ready item 计算必须尊重 blocking graph，而不是只看 owner 或 status
- runtime task 完成后可以更新 work item，但不得把两者 collapse 成同一对象

## Expected Persistent Effects

- work board 状态可被多个 worker 或语义等价观察者一致读取
- 依赖关系、owner 与完成状态在刷新或恢复后仍可重建
- 若存在 task-to-work 映射，该映射在 task terminal 后仍可追溯

## Allowed Variance

- work board 可以本地持久化、内存复制加 durable snapshot，或使用远端协作存储
- ready set 可以显式计算，也可以按查询时动态推导
- 一个 work item 可以不立即触发 runtime task

## Failure Conditions

- work item 与 runtime task 共用同一 identity 和状态机
- blocked item 在依赖未解除时仍被当成 ready item 分配
- claim/release/reassign 无法稳定改变 owner
- runtime task 终态直接覆盖 work item 语义，导致协作层状态丢失
