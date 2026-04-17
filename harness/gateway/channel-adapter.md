# Channel Adapter

## 职责

`ChannelAdapter` 是 gateway 最外侧的协议适配层。

它直接贴着具体接入渠道工作，例如：

- CLI
- Telegram
- Slack
- WebSocket
- Webhook
- Local UI

它的职责不是运行 agent，也不是做 session restore，而是：

- 接收某个 channel 的原始输入
- 把原始输入转成 gateway 可消费的标准 input
- 把 gateway/harness 产出的标准 egress 投影回该 channel
- 处理该 channel 自己的协议、鉴权、消息 ID、回调和重试

## 稳定接口

推荐最小接口：

```text
ChannelAdapter
  - channel_type
  - connect()
  - close()
  - receive_raw() -> raw_channel_message
  - normalize_inbound(raw_channel_message) -> inbound_envelope
  - project_outbound(egress_event) -> channel_event
  - send(channel_event)
```

建议补充：

```text
ChannelIdentity
  - channel_type
  - tenant_id?
  - user_id?
  - conversation_id?
  - device_id?

InboundEnvelope
  - channel_identity
  - input_kind: user_message | attachment | supplement_input | control | resume | wake
  - payload
  - delivery_metadata
```

## 设计要求

- `ChannelAdapter` 必须位于 gateway 外缘，而不是 harness 内部
- 不同 channel 的协议差异应止步于 adapter，不应渗透到 harness
- adapter 应支持 ingress 和 egress 双向投影
- adapter 应支持 progress-like runtime event 的 channel-native 呈现
- adapter 可以内嵌在 gateway 进程中，也可以是远端服务
- adapter 不应直接承担 agent turn 推进职责

## 默认实现映射

当前仓库里的默认实现还没有把 `ChannelAdapter` 作为独立源码目录完全抽出，但已有清晰映射：


## 规范结论

- `ChannelAdapter` 属于 gateway 侧，不属于 harness
- `ChannelAdapter` 是协议边缘，不是运行时控制面
- terminal、GUI、Telegram、Slack、API client 都应被视为不同的 channel adapter 落地
