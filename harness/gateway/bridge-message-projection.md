# Bridge Message Projection

## 职责

`BridgeMessageProjection` 负责把内部 runtime 消息与 gateway egress / input 协议互相投影。

它解决：

- inbound parsing
- message eligibility filtering
- echo / replay dedup
- control_request / control_response 路由

它不直接处理具体 channel 协议格式；那属于 `ChannelAdapter`。

## 稳定接口

```text
BridgeMessageProjection
  - normalize_inbound(raw_message) -> normalized_inbound
  - filter_outbound(runtime_message) -> boolean
  - dedup_inbound(message_id) -> boolean
  - handle_control_request(request) -> control_response
```

## 设计要求

- control flow 与 normal message flow 必须分层
- tool progress 应作为标准 runtime egress event 被可投影地保留
- outbound filtering 必须允许屏蔽内部 chatter
- inbound dedup 必须支持 echo 与 replay 两类场景
- message projection 必须与 transport 抽象分开
- message projection 必须与 channel adapter 分开

## 默认实现映射


## 规范结论

- gateway/bridge 必须显式建模消息投影层
- message projection 位于 gateway 内部，不应下沉到 harness
