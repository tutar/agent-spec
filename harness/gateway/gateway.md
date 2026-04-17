# Gateway

## 职责

`Gateway` 负责把外部 channel 与内部 harness worker 连接起来。

它位于 `ChannelAdapter` 与 `Harness` 之间，负责：

- channel-agnostic input normalization
- message normalization
- control routing
- session binding
- egress projection

当前仓库里的默认实现主要是 remote-control bridge，但规范不应把它限制为单一渠道。

在当前产品语义下，`Gateway` 还承担一个更高层约束：

- `1 Agent = 1 Gateway`
- `1 Gateway = N ChannelAdapterInstance`
- `1 Gateway = N Chat`
- `1 Gateway = N Session`
- `1 Gateway = N HarnessInstance`

## 稳定接口

推荐最小接口：

```text
Gateway
  - register_channel(adapter)
  - receive_input(inbound_envelope) -> normalized_input
  - route_control(control_message) -> control_result
  - bind_session(channel_identity, session_identity) -> binding
  - project_egress(runtime_event) -> egress_event
```

推荐补充：

```text
NormalizedInputMessage
  - channel
  - conversation_id
  - sender_id
  - message_id
  - content
  - attachments
  - metadata

ControlEnvelope
  - subtype
  - request_id
  - payload

ChatSessionBinding
  - gateway_id
  - channel_instance_id
  - chat_id
  - session_id
  - agent_id
```

推荐补充身份对象：

```text
GatewayIdentity
  - gateway_id
  - agent_id

ChannelAdapterInstance
  - channel_instance_id
  - channel_type
  - gateway_id

ChatIdentity
  - chat_id
  - channel_instance_id
  - external_conversation_id

HarnessInstance
  - harness_instance_id
  - agent_id
  - gateway_id
  - session_id?
  - status
```

## 设计要求

- gateway 不等于具体 transport
- gateway 不等于 channel adapter
- gateway 不等于 harness loop
- gateway 必须支持 input normalization
- gateway 必须支持 control flow 与 normal message flow 分层
- gateway 必须支持多渠道扩展
- gateway 默认采用 `1 chat = 1 session` 的 session-per-chat 语义
- 一个 chat 只能归属一个 channel instance
- 一个 chat 只能绑定一个 session
- 一个 gateway 可以同时挂多个同类型 channel instance，也可以同时挂多个异构 channel
- gateway 是 agent 对外的唯一接入总线，不应再被降格成单一 bridge 变体
- gateway 直接管理的是 `HarnessInstance`，而不是 `AgentRuntime`
- `supplement_input` 属于 input，不属于 control
- `interrupt` 属于显式 control，不与 supplement input 混用
- `tool_progress` 应作为标准 runtime egress event 由 gateway 投影
- egress projection 必须允许过滤内部 chatter，只暴露合适的外部事件
- 周期性 due-job ticking 不属于 gateway 核心职责

## 默认实现策略

推荐分为四层：

1. `ChannelAdapter`
   负责接具体外部渠道并做协议适配
2. `InputNormalization`
   负责把外部输入转成内部消息
3. `ControlRouting`
   负责处理 interrupt / permission-response / mode-change 等控制流
4. `SessionBindingAndProjection`
   负责 channel 与 session / harness instance 的绑定及输出投影

## 与其它模块的边界

- 不等于 `Session`
- 不等于 `Harness`
- 不等于 `Transport`
- 不等于 `Orchestration`
- 不等于 `ChannelAdapter`
- `Gateway` 与 `Agent` 在产品语义上是 `1:1`
- `Gateway` 不直接拥有 `AgentRuntime`；`AgentRuntime` 属于 `HarnessInstance` 内部

它位于 `ChannelAdapter` 与 `Harness` 之间，承担统一入口边界。

## 默认实现映射

当前仓库中的默认实现主要是 remote-control 变体：


## 规范结论

- bridge 在规范层应被提升为 gateway
- remote-control 只是当前默认实现，不应限制规范抽象
- 所有外部 chat/channel 接入都应优先复用这层网关边界
- harness 不应内建各 channel 的服务端逻辑
- 一个 gateway 应能同时管理多个 channel、chat、session 与 harness instances
