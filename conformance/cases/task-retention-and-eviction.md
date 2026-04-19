# Case: Task Retention And Eviction

## 目标

验证 task 的 terminal notification 去重、观察者持有与回收边界。

## Preconditions

- runtime 支持 task notification 或语义等价投影
- task 可以被外部观察者持有，例如 detail view、transcript view 或 restore consumer
- runtime 支持 terminal task 的延迟清理、grace period 或等价 eviction 策略

## Ingress

1. 启动一个 task 并观察运行中 progress
2. 让 task 进入 terminal status
3. 在终态后保持一个观察者继续持有该 task
4. 验证持有期间的 notification 与回收行为
5. 释放观察者，再验证 task 可以进入回收路径

## Expected Lifecycle

- progress notification 与 terminal notification 明确分离
- 一个 terminal task 的最终通知只稳定投影一次
- 被观察者持有的 terminal task 不会立即回收
- 持有释放后，task 才允许按 grace period、`evict_after` 或等价策略进入回收

## Expected Runtime Semantics

- `notified` 或等价状态只表示 terminal notification 已稳定提交
- kill path、finally path、restore replay 不会导致双发 terminal notification
- foreground/background 切换或 observer 附着/脱离不改变 task identity
- 回收是资源管理动作，不会改写已经投影给外部的 terminal 事实

## Allowed Variance

- 观察者可以是 UI 组件、session restore consumer、notification queue consumer 或等价对象
- 回收可以分为“task state 驱逐”和“output store 清理”两个阶段
- grace period 的具体时长不是规范要求，只要求边界语义一致

## Failure Conditions

- terminal task 在仍被观察者持有时被立即删除
- 同一 terminal task 向外部重复发出完成或失败通知
- foreground/background 切换导致 task identity 变化
- 回收行为导致 terminal cause、最终结果或输出引用不可再追溯
