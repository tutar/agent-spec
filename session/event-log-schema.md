# Session Event Log Schema

## 目标

本文件定义 `Session` 的事件日志对象。

它关注 durable event log 的接口语义，而不是某种具体文件格式。

## SessionEvent

用于表达 session 中的一条持久化事件。

```text
SessionEvent
  - event_id
  - session_id
  - parent_event_id?
  - event_type
  - timestamp
  - agent_id?
  - task_id?
  - payload
```

推荐 `event_type` 至少包括：

- `user_message`
- `assistant_message`
- `system_message`
- `attachment`
- `tool_result_reference`
- `compact_boundary`
- `branch_link`
- `resume_marker`
- `task_notification`
- `memory_link`
- `metadata_update`

约束：

- 进入 `SessionEvent` 的内容必须是 durable 语义，而不是纯 UI 临时态
- 高频 `tool_progress`、spinner、ephemeral progress 默认不应直接进入 durable event log
- 若工具结果被外存化，transcript/event log 中应持有稳定引用，而不是要求重新生成原结果

## EventSelector

用于表达读取 event log 的选择器。

```text
EventSelector
  - cursor?
  - range?
  - window?
  - event_types?
  - branch?
```

## SessionCheckpoint

用于表达一次 durable append 的提交位置。

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
  - branch_id?
  - event_count?
```

## WakeRequest

用于表达从 durable session 恢复 runtime 的请求。

```text
WakeRequest
  - session_id
  - target_branch?
  - restore_mode?
  - cursor?
  - pending_action_ref?
```

## ResumeSnapshot

用于表达从 event log 恢复后的运行时快照。

```text
ResumeSnapshot
  - session_id
  - wake_id
  - checkpoint
  - runtime_state
  - working_state
  - transcript_slice
  - pending_action?
  - short_term_memory?
  - memory_refs?
  - child_refs?
  - branch_refs?
```

## RuntimeState

`runtime_state` 推荐至少包括：

```text
RuntimeState
  - active_branch?
  - last_checkpoint
  - resume_cursor?
  - pending_continuation?
  - compact_boundary_ref?
```

## WorkingState

`working_state` 推荐至少包括：

```text
WorkingState
  - active_messages?
  - content_replacements?
  - tool_execution_state?
  - context_window_state?
```

## ShortTermMemoryCoverageBoundary

用于表达 short-term memory 覆盖到 transcript 的哪个位置。

```text
ShortTermMemoryCoverageBoundary
  - session_id
  - covered_until_event_id?
  - covered_until_cursor?
  - generated_at
```

约束：

- `ShortTermMemoryCoverageBoundary` 必须能与 `SessionCheckpoint` 对齐
- `wake()` 时若需要消费 short-term memory，应能判断其是否覆盖到待恢复 checkpoint

## 规范结论

- `SessionEvent`、`EventSelector`、`WakeRequest`、`ResumeSnapshot` 应作为 session event log 的标准对象
- `SessionCheckpoint` 应作为 append/read/resume 之间的标准衔接对象
- event log 接口应独立于 JSONL、数据库行或消息数组实现
- `ResumeSnapshot` 必须足以支撑 compact 后恢复，而不依赖旧进程内状态
- branch / sidechain / persisted tool result reference 应通过 durable event 语义进入 session 边界
