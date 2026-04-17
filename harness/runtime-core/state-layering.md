# State Layering

## 目标

本文档定义 `Harness` 相关的状态分层原则。

规范的目标不是复刻某个实现中的字段集合，而是稳定以下边界：

- 哪些状态属于进程级运行态
- 哪些状态属于交互/UI 状态
- 哪些状态应由 session 持久化
- 哪些状态应由具体领域对象自行管理

## 一、为什么需要状态分层

agent 系统通常会同时存在以下几类状态：

- 进程当前的运行环境与缓存
- 当前客户端界面和交互状态
- 当前 session 的 durable history 和生命周期
- tasks、plugins、mcp 等领域对象自己的状态

如果把这些状态都混进一个总状态树，会导致：

- durable 与 non-durable 状态边界模糊
- UI 状态污染 runtime 语义
- 领域对象被迫依赖一个巨型中央 store
- 跨语言实现难以对齐

因此规范上应显式采用状态分层。

## 二、ProcessState

`ProcessState` 是当前 harness 进程的运行态。

它负责：

- 保存当前进程级路径与环境信息
- 保存当前进程级缓存
- 保存当前进程级计量与统计
- 保存当前进程级 feature latches 与临时标志

它不负责：

- 作为 session durable source of truth
- 作为 UI 状态树
- 直接作为外部可观察 lifecycle 信号

推荐特征：

- 进程内单例或等价全局上下文
- 允许重建
- 允许丢失后由 session 恢复必要部分
- 不应假定跨进程共享

## 三、InteractionState

`InteractionState` 是客户端交互层状态。

它负责：

- 当前界面布局和选择状态
- 权限交互上下文
- 当前通知、提示和局部视图状态
- 当前已激活的会话级能力视图

它不负责：

- 持久化 transcript
- 承担 session restore
- 成为所有领域对象的唯一真实来源

推荐特征：

- 面向当前客户端
- 支持订阅式更新
- 不要求 durable
- 可针对不同前端形态自由实现

## 四、DomainState

`DomainState` 是各子系统自己的状态模型。

例如：

- task state
- plugin state
- mcp connection state
- notification queue state

规范要求：

- 领域对象应有自己的状态边界
- 不应为了统一而强行压成一个模糊总状态对象
- 领域状态可以投影到 `InteractionState`，但不应与之等价

## 五、与 Session 的边界

- `Session`
  负责 durable history、restore 和 lifecycle semantics
- `Harness State Layering`
  负责区分进程态、交互态和领域态

推荐约束：

- 能从 session 恢复的状态，不应只存在于 `ProcessState`
- 纯 UI / interaction 状态，不应进入 session durable store
- `SessionLifecycleState` 应单独抽象，不应埋进 `InteractionState`

## 六、推荐抽象

```text
ProcessState
  - environment
  - runtime_caches
  - counters
  - feature_latches
```

```text
InteractionState
  - view_state
  - selection_state
  - permission_interaction_state
  - notification_state
  - active_capability_views
```

```text
DomainStateRegistry
  - tasks
  - plugins
  - mcp
  - other_subsystems
```

## 七、默认实现映射

本仓库当前可映射出一套符合该规范的默认实现：

### ProcessState


默认实现特征：

- 进程级全局单例
- 保存 session identity、路径、缓存、telemetry、feature latches 等运行态

### InteractionState


默认实现特征：

- 使用订阅式 in-memory store
- 驱动 CLI/Ink 界面与当前交互状态

### DomainState

- task state
- plugin / mcp projections

## 八、规范结论

- `ProcessState`、`InteractionState`、`DomainState` 应显式分层
- 规范应稳定分层边界，而不是绑定具体字段表
- `SessionLifecycleState` 应单独归属 `Session`，不应并入 harness 内部状态
