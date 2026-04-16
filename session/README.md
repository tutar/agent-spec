# Session

## 职责

`Session` 是 agent runtime 的 durable state boundary。

模块总览见 [../module-overview.md](../module-overview.md)，术语归属见 [../terminology-and-ownership.md](../terminology-and-ownership.md)。

它负责：

- 标识一条可持续运行的 agent 会话
- 维护 runtime state
- 持久化 append-only transcript
- 恢复可继续运行的 working state
- 挂载 session 内的短期连续性记忆
- 串联 durable memory、subagent branch、remote session 等外部引用

它也必须跨部署形态稳定：

- `Local`
  通常是本地 durable transcript + restore
- `Cloud`
  通常是远端 event log / restore store

它也是 `resume` 的真源：

- `Local` 下应用或系统重启后，应仍可基于本地 durable transcript / working state 恢复
- `Cloud` 下 harness 或 sandbox 实例被替换后，应仍可基于远端 event log / restore store 恢复

规范上必须明确：

- transcript 不是 memory
- compact summary 不是 transcript
- `CLAUDE.md` / memory files 不是 session 本身
- agent memory 不是主 session transcript

本规范中的 `Session` 同样采用 `Local-first` 抽取方式：

- 默认实现通常是本地 append-only transcript 与 working-state restore
- 但稳定接口不得要求本地文件系统、单机场景或某一种持久化介质
- `Local` 与 `Cloud` 的差异只能体现在部署位置和实现技术，不应改变 `resume` 语义

## 稳定接口

推荐最小接口：

```text
SessionStore
  - create_session(session_metadata) -> session_id
  - append_events(session_id, events) -> session_checkpoint
  - get_session(session_id) -> session_metadata
  - get_events(session_id, selector) -> event_slice
  - get_runtime_state(session_id) -> runtime_state
  - get_working_state(session_id) -> working_state
  - wake(session_id) -> resumed_session_handle
```

这些接口表示可恢复 durable state boundary 的最小契约，不要求各语言实现暴露同名 API。

推荐标准对象：

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
```

```text
ResumedSessionHandle
  - session_id
  - wake_id
  - resume_snapshot
```

若实现了 session 内记忆层，还应补：

```text
SessionMemoryStore
  - load_short_term_memory(session_id)
  - update_short_term_memory(session_id, transcript_delta)
  - list_memory_links(session_id)
```

推荐补充：

```text
SessionLifecycleController
  - get_state(session_id) -> lifecycle_state
  - set_state(session_id, lifecycle_state, details?)
  - subscribe(session_id, listener) -> lifecycle_event_stream
```

若实现支持 branch / sidechain，还应补：

```text
SessionBranchStore
  - create_branch(parent_session_id, branch_metadata) -> branch_ref
  - list_child_refs(session_id) -> child_refs
  - resolve_branch(ref) -> branch_handle
```

### 核心能力

宣称实现 `Session` 模块时，至少应具备：

- append-only event log 或语义等价机制
- session identity
- checkpoint / cursor
- runtime / working state restore
- wake / resume
- 结构化 lifecycle state
- compact boundary 后可继续恢复

### 标准扩展

推荐把下列能力作为 `Session` 标准扩展实现：

- `ShortTermSessionMemory`
  - continuity summary
  - away summary
  - compact-aware continuation
- `DurableMemoryLinkage`
  - recall
  - scoped memory linkage
  - project / user / agent / local scope
- `MemoryConsolidation`
  - post-turn consolidation
  - background consolidation
  - dream-style consolidation
- `BranchAndSidechain`
  - subagent transcript
  - teammate transcript
  - sidechain transcript
  - remote session link

这些扩展的重点是恢复语义与外部可观察行为，而不是某一种 summary 或 recall 算法。

## 状态分层

`Session` 规范必须至少区分三层状态：

- `SessionLifecycleState`
  - `idle / running / requires_action`
  - 面向 gateway、UI、remote control、push 系统等外部消费者
  - 不用于重建内部执行上下文
- `RuntimeState`
  - durable 的执行进度状态
  - 例如当前 checkpoint、active branch、pending tool continuation、resume cursor
  - 供 `wake()` / `resume` 读取
- `WorkingState`
  - 恢复当前 turn 或 continuation 所需的结构化运行上下文
  - 例如 compact 后的当前消息窗口、pending action binding、tool result replacement state
  - 不要求外部系统直接消费

约束：

- `LifecycleState` 不能替代 `WorkingState`
- `WorkingState` 不能通过扫描 transcript 文本临时推导来代替 durable restore
- `RuntimeState` 与 `WorkingState` 可以落在同一存储里，但语义上必须可区分

如果本地宿主希望提供更简单的 API，
例如 `emit_event()` 或直接读取完整 transcript，
也应把它们视为 `append_events()` / `get_events()` 的 convenience wrapper，
而不是新的 session 语义。

## 默认实现

当前代码库中的默认 session 实现由这些部分组成：

- [bootstrap/state.ts](../../cc/bootstrap/state.ts)
  identity
- [utils/sessionState.ts](../../cc/utils/sessionState.ts)
  runtime state
- [utils/sessionStorage.ts](../../cc/utils/sessionStorage.ts)
  transcript + sidechain storage
- [utils/sessionRestore.ts](../../cc/utils/sessionRestore.ts)
  restore
- [services/SessionMemory/sessionMemory.ts](../../cc/services/SessionMemory/sessionMemory.ts)
  short-term session memory
- [memdir/findRelevantMemories.ts](../../cc/memdir/findRelevantMemories.ts)
  durable memory recall linkage
- [tools/AgentTool/agentMemory.ts](../../cc/tools/AgentTool/agentMemory.ts)
  agent-scoped durable memory

默认落点上：

- 主 transcript 用 append-only JSONL 持久化
- subagent 使用 sidechain transcript
- session memory 使用 session 级 continuity summary
- durable memory 通过 recall/linkage 接入，而不承担 restore 责任

这里的 JSONL、sidechain 文件与当前恢复流程只代表 `cc` 的本地默认实现，不构成跨语言 SDK 的介质要求。

## 要解决的问题

- 如何让会话在 harness 崩溃后仍可恢复
- 如何把当前 prompt window、raw transcript、working state、session memory 区分开
- 如何在长会话中保留 continuity，而不依赖全量重放 transcript
- 如何把 compact boundary、resume snapshot 与 short-term memory 的关系稳定下来
- 如何把 memory consolidation 与 session restore 解耦但保持联动
- 如何把跨会话 durable memory 接进来，但不与 session restore 混淆
- 如何兼容 sidechain transcript、subagent transcript、remote session 和 task state
- 如何让本地 session 与远端 session 共享同一语义边界

## 核心边界

- `Session`
  管 durable state、restore、memory linkage
- `Harness`
  读取 session 切片并构造当前 model input
- `ContextProvider`
  把 durable memory / project memory 注入当前 turn
- `ContextGovernance`
  决定 compact，不等于 session 自身
- `SessionLifecycleState`
  对外暴露 `idle / running / requires_action` 语义，不等于 transcript 或 UI state

### `compact boundary` 与恢复的关系

`compact boundary` 进入 transcript 后，`Session` 仍必须保持以下恢复语义：

- transcript 仍然是恢复真源
- short-term memory 只能作为 compact 后 continuity 的辅助对象，不能取代 transcript
- `ResumeSnapshot` 必须明确指出当前恢复是基于哪个 checkpoint / compact boundary 构建的
- 若 short-term memory 尚未稳定，`wake()` 应显式返回等待、超时或使用最近稳定摘要的语义，而不能静默漂移

换句话说：

- compact 后允许不再重放全部历史消息
- 但不允许失去对 compact 前 durable history 的可追溯性

session 域内的 memory 详细模型见 [memory-model.md](memory-model.md)。

## 子代理与 remote session

session 体系必须兼容：

- sidechain transcript
- subagent transcript
- teammate transcript
- remote agent session / task link

要求：

- 子对象必须可追溯到父 session
- 子对象可以独立落盘
- restore 时必须恢复引用关系
- 必须区分 durable sidechain 与 ephemeral fork

推荐最小对象：

```text
BranchRef
  - ref_id
  - parent_session_id
  - branch_id
  - kind: sidechain | subagent | teammate | remote_session | ephemeral_fork
  - transcript_ref?
  - created_at
  - metadata?
```

其中：

- `ephemeral_fork`
  可以不要求 durable transcript
- 其余 branch / sidechain 类型
  恢复时必须能重新建立父子关系与 transcript 引用

## 目录内文档

- [event-log-schema.md](event-log-schema.md)
- [lifecycle-state.md](lifecycle-state.md)
- [memory-model.md](memory-model.md)
- [memory-consolidation.md](memory/memory-consolidation.md)
- [dream-consolidation.md](memory/dream-consolidation.md)

## 规范结论

- `Session` 是 durable state boundary，不是 message array
- `Session` 模块主要负责 transcript、restore、working state 和 session memory linkage
- 长短期记忆的详细规范应下沉到独立文档，而不是挤在总览页里
- session 语义不应因为 Local、Cloud 的落盘位置不同而漂移
- session 应作为 harness / sandbox 可替换后的恢复真源
- `append_events()` 与 `session_checkpoint` 应优先于进程内 transcript 变异
- direct-call session API 如存在，也必须严格由 event log 语义推导
- short-term memory、durable memory linkage、memory consolidation 与 branch transcript 应优先作为标准扩展定义
