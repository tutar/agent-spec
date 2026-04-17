# Failure And Terminal States

## 目标

本文档统一 agent runtime 中几类最容易分叉的语义：

- terminal state
- error class
- retryability
- cancellation
- requires_action

如果这些语义不统一，Local 和 Cloud 很快就会演化出不同的完成条件与错误面。

## 一、核心原则

agent 应区分三层东西：

- `event`
  运行过程中的增量事实
- `terminal state`
  某个执行单元的结束方式
- `error class`
  导致失败或阻塞的原因类型

它们不应混成一个自由文本错误对象。

## 二、统一终止态

推荐最小终止态：

```text
TerminalState
  - completed
  - failed
  - cancelled
  - requires_action
  - aborted
  - timed_out
```

推荐语义：

- `completed`
  正常结束，且已产生稳定结果
- `failed`
  因错误结束，且该执行单元未能完成其语义目标
- `cancelled`
  因显式取消而结束，通常由用户、父任务或 sibling failure 触发
- `requires_action`
  非真正“结束”，但当前执行单元已停在外部动作边界
- `aborted`
  因 runtime 切换、fallback、transport 中断或宿主关闭而中止，通常带恢复语义
- `timed_out`
  因超时策略结束，是否可恢复取决于 error class 和 owner policy

## 三、统一错误分类

推荐最小错误分类：

```text
AgentErrorClass
  - auth_error
  - permission_denied
  - rate_limited
  - context_too_large
  - output_limit
  - validation_error
  - tool_protocol_error
  - dependency_unavailable
  - network_transient
  - execution_target_failed
  - conflict
  - not_found
  - internal_error
  - non_retryable
```

这些分类的目的不是穷举所有供应商或工具错误，
而是给 runtime / orchestration / session 提供足够稳定的控制语义。

## 四、错误对象

推荐标准对象：

```text
AgentError
  - class: AgentErrorClass
  - message
  - retryable: boolean
  - terminal: boolean
  - source: model | tool | task | session | sandbox | transport | policy
  - code?
  - details?
```

约束：

- `retryable` 必须显式，而不是靠 message 猜测
- `terminal` 表示当前执行单元是否必须结束
- `source` 用于定位责任边界，不等于 error class

## 五、RequiresAction 与 Failed 的边界

`requires_action` 不是一种失败。

它表示：

- runtime 已到达稳定阻塞点
- 需要外部输入、审批、选择或恢复动作
- 当前状态应可持久化并可恢复

因此：

- policy ask 不应直接映射成 `failed`
- approval pending 不应映射成 `cancelled`
- plan review / human input 不应被编码成普通 error

## 六、Cancelled 与 Aborted 的边界

这两个状态必须区分：

- `cancelled`
  表示有明确发起方终止了执行
- `aborted`
  表示当前执行上下文被打断或替换，通常不说明“意图取消”

示例：

- 用户点停止: `cancelled`
- sibling tool 失败后取消兄弟执行: `cancelled`
- streaming fallback 后丢弃旧 executor: `aborted`
- desktop runtime 退出但 session 可恢复: `aborted`
- cloud worker 丢失租约后停止本地执行: `aborted`

## 七、Retryability 规则

`retryable` 应作为标准字段，而不是某层内部策略。

建议最小规则：

- `network_transient`
  通常 `retryable=true`
- `rate_limited`
  通常 `retryable=true`
- `execution_target_failed`
  取决于是否允许 reprovision
- `context_too_large`
  通常不直接 retry，而是先 compact/rewrite
- `permission_denied`
  通常 `retryable=false`，除非外部审批后重试
- `validation_error`
  通常 `retryable=false`
- `non_retryable`
  必须 `retryable=false`

是否自动重试，应由 owner policy 决定；
但是否“可被重试”必须是标准语义。

## 八、各层对齐要求

### Runtime

- terminal state 必须稳定投影给上层 agent client
- model/provider error 必须先归一化，再参与状态机决策
- `requires_action` 应作为 terminal-like stop reason 暂停 turn

### ToolExecutor

- 工具失败应优先发 `tool_failed` 事件，并附 `AgentError`
- 工具取消应优先发 `tool_cancelled` 事件，而不是伪装成失败
- sibling error 导致的停止应保留取消来源

### TaskManager

- task terminal state 应与 runtime terminal state 术语兼容
- `killed` 可视为 task 侧的 `cancelled`
- task output store 不应吞掉 terminal cause

### Session

- terminal state 与关键错误应能 durable 落盘
- `requires_action` 必须以结构化对象进入 session 可恢复边界
- restore 后应能重建最近一次 terminal-like stop 原因

## 九、事件与终止态的关系

事件流建议至少覆盖：

```text
started
progress
completed
failed
cancelled
requires_action
aborted
timed_out
```

但注意：

- event 是过程事实
- terminal state 是结束归类

一个执行单元可以发出很多 `progress` 事件，
最终只对应一个 terminal state。

## 十、Convenience API 约束

如果本地宿主提供 direct-call API，
它返回的异常、结果和完成状态也必须能从 core event stream 严格推导。

不允许 direct-call API：

- 发明 event stream 中不存在的错误分类
- 把 `requires_action` 静默吞成普通返回值
- 把 `aborted` 和 `cancelled` 混成同一个结果
- 把可恢复错误包装成不可恢复异常

## 规范结论

- terminal state、error class、retryable 应成为统一稳定语义
- `requires_action` 不是失败，`cancelled` 不等于 `aborted`
- 任何层级的 direct-call facade 都不得改变 core streaming contract 的错误与结束语义
- 如果某个实现无法稳定表达这些状态，它就还不适合成为 agent 的核心实现边界
