# Bridge Transport

## 职责

`BridgeTransport` 负责 bridge/gateway 与远端会话连接之间的数据平面抽象。

它不关心 session 创建策略，也不关心 message normalization，只关心连接、写入、读取、关闭、恢复和少量状态上报。

## 稳定接口

```text
BridgeTransport
  - connect()
  - close()
  - write(event)
  - write_batch(events)
  - on_data(callback)
  - on_close(callback)
  - on_connect(callback)
  - flush()
  - get_resume_cursor()
```

推荐补充：

```text
BridgeTransportStatus
  - connected
  - reconnecting
  - closed
```

## 设计要求

- transport 抽象必须与 bridge session lifecycle 分开
- transport 必须支持 resume cursor / replay dedup 所需信息
- transport 写入与读取协议可以不同
- transport 应允许独立上报 state / metadata / delivery

## 默认实现映射

- 当前默认实现包括：
  - `HybridTransport`
  - `SSETransport + CCRClient`

## 规范结论

- transport 是 bridge/gateway 的子层，不应和 gateway 本身混写
