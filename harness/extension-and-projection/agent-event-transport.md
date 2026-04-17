# Agent Event Transport

## 职责

`AgentEventTransport` 负责把 runtime 内部的关键状态变化投影给外部 agent client。

它独立于主消息流，面向：

- session lifecycle
- task lifecycle
- bg task / subagent UI
- turn completion / idle detection

## 稳定接口

```text
AgentEventTransport
  - publish(event)
  - drain() -> events

AgentRuntimeEvent
  - type
  - subtype
  - session_id
  - uuid
  - payload
```

推荐事件：

- `task_started`
- `task_progress`
- `task_notification`
- `session_state_changed`

## 设计要求

- 不能与 transcript 混用
- 不能与模型消息流混用
- 允许多个子系统写入
- 允许外部客户端单独消费
- 默认可为 best-effort queue，但语义应稳定

## 默认实现取向

该通道可以实现为内存队列、IPC event bus、远端事件流或 UI projection adapter，只要事件语义保持稳定即可。

## 规范结论

- agent client 应有独立系统事件通道
- runtime state projection 不应强塞进主消息流
- 本地 direct-call API 可以聚合这些事件，但不应创造事件通道之外的独有语义
