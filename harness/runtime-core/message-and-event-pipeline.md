# Message And Event Pipeline

## 目标

本文档定义 `Harness` 内部消息管线与对外事件管线的分层。

规范目标不是绑定某个消息类型定义文件，而是稳定以下边界：

- runtime 内部消息对象如何建模
- 流式增量事件如何建模
- 对外事件/消息协议如何投影
- 哪些控制消息不应直接暴露给外部客户端

## 一、为什么需要分层

agent harness 同时面对三类问题：

- turn 内部的语义推进
- 模型流式返回的运输过程
- 对外客户端消费的稳定协议

如果把这三类对象混成一种消息，会导致：

- transcript 与 streaming 增量混淆
- 内部控制消息泄漏到外部客户端
- tool use / tool result 一致性难以维护
- fallback / compact / tombstone 难以表达

因此规范上应至少区分：

- `InternalMessage`
- `StreamEvent`
- `RuntimeEvent` 或等价的外部事件对象

## 二、InternalMessage

`InternalMessage` 是 harness 内部的语义对象。

它负责：

- 构成 turn 的主消息链
- 记录 assistant / user / system 语义
- 承载 progress、attachment、boundary、control message
- 进入 transcript 或参与 transcript 派生逻辑

推荐至少支持的语义类别：

- `assistant`
- `user`
- `system`
- `progress`
- `attachment`
- `boundary`
- `control`

其中：

- `boundary`
  用于 compact / snip / microcompact 等语义边界
- `control`
  用于 tombstone、interruption、internal repair 等运行时控制信号

## 三、StreamEvent

`StreamEvent` 是流式传输层对象。

它负责：

- 表达 request start
- 表达 assistant delta
- 表达 message stop / message delta 等流式中间态
- 为客户端提供低延迟 streaming 体验

它不等于：

- durable transcript entry
- 最终 assistant message
- 对外稳定业务事件

推荐最小类型：

```text
StreamEvent
  - request_started
  - assistant_delta
  - message_delta
  - message_stop
```

## 四、External Runtime Event

`ExternalRuntimeEvent` 是对外消费的稳定协议对象。

它负责：

- 给 agent / IDE / viewer 暴露稳定事件流
- 屏蔽内部消息实现细节
- 将内部 boundary / task / requires_action 等语义投影为稳定外部事件

推荐直接复用 [object-model.md](../../object-model.md) 中的 `RuntimeEvent`。

其中与失败、取消、阻塞相关的事件字段应与
[failure-and-terminal-states.md](failure-and-terminal-states.md) 保持一致。

## 五、推荐分层关系

```text
UserInput
  -> InternalMessage
  -> AgentRuntime
  -> StreamEvent
  -> InternalMessage / ControlMessage
  -> ExternalRuntimeEvent
```

约束：

- `InternalMessage` 是 runtime 的语义源
- `StreamEvent` 是 transport 层增量对象
- `ExternalRuntimeEvent` 是对外稳定接口

## 六、Tool Use / Tool Result 一致性

消息管线必须显式维护 `tool_use` / `tool_result` 一致性。

最小要求：

- 每个 `tool_use` 必须能关联到对应 `tool_result`
- streaming abort / fallback 后不得遗留孤儿 `tool_use`
- repair / abort 时允许生成结构化控制结果或 synthetic result
- 外部事件层不应破坏内部配对关系

## 七、Boundary And Control Messages

规范上应允许 harness 内部存在不直接暴露给外部客户端的控制消息。

典型场景：

- compact boundary
- context collapse boundary
- tombstone
- interruption
- repair placeholder

这些消息：

- 可以存在于内部消息层
- 可以进入 transcript 或影响 transcript
- 但不一定应原样出现在外部 agent 协议中

## 八、投影规则

推荐规则：

- `InternalMessage`
  面向 runtime 和 transcript
- `StreamEvent`
  面向 streaming transport
- `ExternalRuntimeEvent`
  面向外部协议

因此实现应支持：

- internal -> external 的筛选和投影
- internal -> transcript 的持久化策略
- stream -> external 的选择性透传

## 九、默认实现映射

本仓库当前可映射出一套符合该规范的默认实现：

### 输入与会话投影边界


默认实现特征：

- 在进入 query loop 前写 transcript
- 持有会话级 `mutableMessages`
- 将内部对象投影为 agent message

### 内部消息工厂与变换


### Turn 级消息推进状态机


默认实现特征：

- turn 内统一推进 `Message`、`StreamEvent`、tool loop、boundary、recovery
- 内部存在 tombstone、attachment、tool_use_summary 等控制对象

## 十、规范结论

- harness 应显式分层 `InternalMessage`、`StreamEvent`、`ExternalRuntimeEvent`
- 对外协议不应等于内部消息对象
- `tool_use / tool_result` 一致性应成为消息管线的核心约束
- boundary 和 control message 应被允许存在，但不要求全部外露
- 外部事件字段应优先复用 [../../object-model.md](../../object-model.md) 中的 canonical objects
