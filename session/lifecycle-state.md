# Session Lifecycle State

## 目标

本文档定义 `Session` 的外部可观察生命周期状态。

这里的 lifecycle state 不等于：

- transcript
- working state
- UI state
- process state

它解决的是：

- 当前 session 是否正在运行
- 当前 session 是否空闲
- 当前 session 是否阻塞在需要外部动作的节点

## 一、为什么需要单独建模

session 持久化解决的是“历史和恢复”。

但对外部系统来说，还需要知道：

- 当前是否正在生成
- 当前是否已经结束本轮工作
- 当前是否在等待批准、选择或输入

如果不把这些显式建模，就会退化成：

- 从 event stream 反推状态
- 从 UI 局部状态猜当前状态
- 用字符串消息近似表达 `requires_action`

因此 `SessionLifecycleState` 必须是一等语义。

## 二、推荐最小状态

```text
SessionLifecycleState
  - idle
  - running
  - requires_action
```

含义：

- `idle`
  当前没有活跃 turn 正在推进
- `running`
  当前正在推进 turn、执行 tool、等待模型流式返回或处理 continuation
- `requires_action`
  当前被阻塞，等待外部系统或用户提供动作

## 三、RequiresAction

`requires_action` 不应只是一个布尔值。

它应携带结构化上下文，至少包括：

```text
RequiresAction
  - tool_name
  - action_description
  - tool_use_id
  - request_id
  - input?
```

这样外部系统才能：

- 展示阻塞原因
- 恢复具体动作上下文
- 直接驱动 approval / question / plan-review UI

`requires_action` 与 `Session restore` 的关系必须明确：

- `requires_action` 是 lifecycle state
- pending action binding 是 `WorkingState` / `ResumeSnapshot` 的一部分
- `wake()` 后若 action 尚未完成，系统必须恢复到同一阻塞语义，而不是仅恢复一条描述文本

## 四、推荐接口

```text
SessionLifecycleController
  - get_state(session_id) -> lifecycle_state
  - set_state(session_id, lifecycle_state, details?)
  - subscribe(session_id, listener)
```

推荐补充：

```text
SessionExternalMetadata
  - permission_mode?
  - pending_action?
  - post_turn_summary?
  - task_summary?
```

## 五、与其它状态的边界

### 与 SessionStore 的边界

- `SessionStore`
  管 durable transcript、event log、restore
- `SessionLifecycleState`
  管当前会话的可观察运行状态

### 与 ProcessState 的边界

- `ProcessState`
  关心当前进程内部状态
- `SessionLifecycleState`
  关心外部可观察的 session 状态

### 与 InteractionState 的边界

- `InteractionState`
  可消费 lifecycle state
- 但 lifecycle state 不应从 UI 状态反推

## 六、推荐语义约束

- lifecycle state 必须能独立于 UI 消费
- `requires_action` 必须带结构化 details
- `idle` 应在真实完成当前 turn 后发出，而不是在中间停顿时误发
- 不同客户端应共享同一 lifecycle 语义，而不是各自猜测
- lifecycle state 不应替代 restore state；恢复时必须同时恢复 pending action 的结构化绑定

## 七、默认实现映射

本仓库当前默认实现映射到：


默认实现特征：

- 最小状态集为 `idle / running / requires_action`
- `requires_action` 具备结构化 details
- lifecycle state 可同步到 external metadata
- 可镜像到 agent event stream

## 八、规范结论

- `SessionLifecycleState` 应成为 `Session` 模块的稳定接口之一
- 它不应被埋进 UI state、process state 或 transcript parsing 逻辑里
- `requires_action` 必须是结构化对象，而不是字符串消息
- `requires_action` 与 `failed/cancelled/aborted` 的边界应与 [../harness/runtime-core/failure-and-terminal-states.md](../harness/runtime-core/failure-and-terminal-states.md) 保持一致
