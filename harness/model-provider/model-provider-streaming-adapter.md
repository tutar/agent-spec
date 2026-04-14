# Model Provider Streaming Adapter

## 目标

本文档定义模型流式协议适配层的职责。

它描述的是：

- 如何把厂商流式协议归一化为 harness 可消费的流式语义
- 如何处理 partial assistant message、usage 回填和 stop reason 回填
- 如何显式表达 streaming fallback

它不描述：

- tool 执行
- task lifecycle
- sandbox execution

## 一、职责

`ModelProviderStreamingAdapter` 负责：

- 消费 provider streaming protocol
- 归一化 content block / delta / stop 语义
- 产出 harness 可消费的 `StreamEvent` 和 message fragments
- 处理 streaming fallback 与 error normalization

## 二、为什么需要单独一层

底层模型厂商流式协议通常有这些特点：

- 事件类型细碎且厂商特有
- usage 和 stop reason 可能后到
- tool call 可能以 block start / delta / stop 形式分多次送达
- 某些失败需要切到 non-streaming fallback

如果这层不被单独抽象，上层 runtime 会直接耦合 provider 事件语义。

provider stream transport、认证与 wire-shape 边界由
`ModelProviderAdapter` 统一吸收。

## 三、推荐最小接口

```text
ModelProviderStreamingAdapter
  - stream(request) -> stream_events
  - normalize_event(provider_event) -> stream_event | message_fragment
  - handle_streaming_fallback(error) -> fallback_decision
```

## 四、必须支持的语义

- `request_started`
- `assistant_delta`
- `message_fragment_completed`
- `message_completed`
- `usage_finalized`
- `stop_reason_finalized`
- `streaming_fallback`

## 五、语义要求

### 1. partial assistant message

适配层必须允许：

- assistant message 先以部分内容出现
- usage、stop reason 在后续事件里回填

### 2. content block normalization

适配层必须统一不同 provider 的 block 粒度差异，至少覆盖：

- text
- reasoning / thinking
- tool use
- provider-specific server tool use

### 3. streaming fallback

`streaming_fallback` 必须是一等语义。

它不应只是一个普通异常，因为上层需要借此：

- tombstone 旧 partial assistant messages
- discard 旧 tool execution state
- 启动 fallback request

## 六、错误归一化要求

适配层应至少区分：

- 可恢复 streaming 错误
- 需要转 non-streaming fallback 的错误
- 不可恢复错误

并给出：

- retryable?
- fallback_to_non_streaming?
- surface_to_runtime?

## 七、与 AgentRuntime 的边界

- `ModelProviderStreamingAdapter`
  只负责 provider stream -> normalized stream
- `AgentRuntime`
  负责 turn 级状态推进、tool loop、fallback 协调

适配层不应直接拥有：

- tombstone 策略
- task state
- tool executor 生命周期

## 八、默认实现映射

本仓库当前默认实现映射到：

- [services/api/claude.ts](../../../cc/services/api/claude.ts)

默认实现特征：

- 归一化 `message_start / content_block_* / message_delta / message_stop`
- 允许 stop reason 和 usage 后到
- 支持 streaming -> non-streaming fallback
- 在进入上层前先做 tool_use/tool_result pairing 修复

## 九、规范结论

- 模型流式协议应通过 `ModelProviderStreamingAdapter` 与 runtime 解耦
- `streaming_fallback` 必须是显式标准语义
- usage 和 stop reason 的后到回填必须被规范允许
