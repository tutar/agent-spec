# Case: Background Agent

## 目标

验证 agent orchestration 中的后台 agent task 语义。

本用例只覆盖“后台 agent 是否被正确 task 化并可被父链路观察”。
更细粒度的 task lifecycle、output cursor、notification dedupe 与 retention/eviction 语义，
分别由 `runtime-task-lifecycle`、`task-output-cursor-and-resume`、`task-retention-and-eviction` 覆盖。

## Preconditions

- runtime 支持 `background agent task`
- task manager 支持状态跟踪
- session 支持在父链路继续推进时接收子 agent 结果或通知

## Ingress

- 一条会触发后台 subagent 的请求

## Expected Lifecycle

- 父 turn 不必阻塞等待子 agent 完成
- 子 agent 获得独立 task handle 或等价对象
- 子 agent 终态至少可区分 `completed / failed / killed`

## Expected Runtime Events

父链路至少应能观测到：

- `task_created` 或等价事件
- `task_progress` 或等价事件
- `task_completed` / `task_failed`

## Expected Persistent Effects

- 子 agent 有独立 output reference、transcript branch 或等价 durable log
- 父 session 可以在子 agent 运行期间继续处理其他 ingress

## Allowed Variance

- 可以是本地后台任务，也可以是 remote background task
- 任务通知可以通过 hook、agent event transport 或 gateway egress 投影

## Failure Conditions

- 后台 agent 实际阻塞父 turn 才能继续
- task 状态不可查询
- 子 agent 输出无法 durable 追踪
