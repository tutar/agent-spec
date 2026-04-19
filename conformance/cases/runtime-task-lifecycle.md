# Case: Runtime Task Lifecycle

## 目标

验证 task 的基础状态机、终态收敛与注册后可观察语义。

## Preconditions

- runtime 支持可查询的 task lifecycle
- task manager 或等价组件支持按 task handle 读取状态
- 至少一种执行对象可被包装为 task

## Ingress

1. 启动一个可被 task 化的执行单元
2. 观察其从注册到运行再到终态的状态变化
3. 分别验证正常完成、执行失败和显式 kill 三条路径中的至少两条

## Expected Lifecycle

- task 至少可区分 `pending / running / completed / failed / killed`
- task 进入终态前可持续追加 progress 或等价运行中事实
- 一个 task 最终只收敛到一个 terminal status
- task 一旦进入 terminal status，不会再回退到 `running`
- `killed` 表示 task 被显式终止

## Expected Runtime Semantics

- task status 变化可被外部观察，而不是只存在于进程内局部对象
- direct-call 或 convenience API 不得绕开同一 lifecycle 语义
- terminal cause 至少可通过 task state、terminal event 或等价 metadata 追溯

## Allowed Variance

- `pending` 可以非常短暂，甚至在某些实现中与 `running` 几乎连续
- progress 可以表现为显式 `task_progress` 事件，也可以是语义等价的运行中更新
- 失败原因可以通过结构化 error、event metadata 或 durable log 暴露

## Failure Conditions

- task 只暴露“运行中/结束”两态，无法区分 `completed / failed / killed`
- 一个 task 在同一次执行中出现多个互相冲突的 terminal status
- kill 后 task 被错误投影为普通失败或仍继续推进
- terminal cause 只能在瞬时日志中看到，无法被恢复或查询
