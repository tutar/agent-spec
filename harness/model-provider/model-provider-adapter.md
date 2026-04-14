# Model Provider Adapter

## 目标

`ModelProviderAdapter` 负责把不同模型厂商、不同 API 风格、不同能力组合，统一适配到 agent harness 可消费的标准语义。

它的职责不是重新发明模型协议，而是把模型差异收敛到一层明确边界内，避免这些差异泄漏到 runtime、tool、policy 和 context governance。

它不负责适配 sandbox、执行环境、VPC 拓扑或凭据代理，这些是另一层接口问题。

## 适配层定位

推荐分层如下：

```text
Business / Product Logic
  -> Agent Harness
  -> Model Provider Adapter
  -> Vendor SDK / HTTP API
  -> LLM Provider
```

要求：

- 上层只感知标准化后的能力与事件
- 下层保留模型厂商特有协议与优化
- 模型切换不应迫使 runtime 重写

provider client / transport / auth 是 `ModelProviderAdapter` 的实现关注点，
不是另一个并列稳定接口。

## Model Provider Adapter 职责

- 发现或声明模型能力
- 标准化请求参数
- 标准化流式输出事件
- 标准化 tool calling 事件
- 标准化 usage / token / reasoning 统计
- 标准化错误类型与可恢复性信号
- 提供原生能力与 harness fallback 的切换入口
- 吸收 provider transport / auth / endpoint / wire protocol 差异

## 标准输入

模型 provider 适配层至少应接受以下逻辑输入：

- 标准化 messages
- 标准化 system instructions
- tool definitions 或 tool references
- 输出约束
- reasoning 或 effort 提示
- 超时、重试、budget 等控制参数

输入接口应表达语义，而不是 vendor 参数名。

例如：

- `max_output`
  而不是某家 API 的专有字段名
- `tool_mode`
  而不是某个协议里的私有枚举
- `response_format`
  而不是绑定某家 JSON mode 的字段

## 标准输出

模型 provider 适配层对上层至少应统一输出：

- 文本 delta
- 完整 assistant message
- tool call 请求
- structured output 结果
- usage 数据
- stop reason
- recoverable / non-recoverable error

## 标准能力视图

能力视图是 runtime 做能力路由的依据，建议至少包含：

- `supports_streaming`
- `supports_native_tool_calling`
- `supports_parallel_tool_calling`
- `supports_structured_output`
- `supports_reasoning_control`
- `supports_server_side_search`
- `supports_prompt_caching`
- `supports_vision_inputs`
- `supports_audio_inputs`

能力视图允许三种来源：

- 静态配置
- provider 元数据
- 启动时探测

## 事件归一化要求

不同模型对流式事件的切分粒度差异很大，适配层必须统一为上层可消费的事件模型。

至少需要统一：

- 文本增量事件
- reasoning 增量事件
- tool call 开始事件
- tool call 参数增量或完整事件
- message 完成事件
- usage 完成事件

如果底层模型不暴露增量粒度，适配层应允许退化为批量消息事件，但语义上仍归一到同一事件类别。

## 错误归一化要求

适配层必须把供应商错误统一为少数几类 runtime 可消费错误：

- 鉴权错误
- 限流错误
- 上下文过长错误
- 输出上限错误
- 工具协议错误
- 网络或暂时性错误
- 非可恢复错误

同时还应给出：

- 是否可重试
- 是否建议 compact/recovery
- 是否建议 fallback model

## 原生能力与 fallback 的边界

适配层应遵循以下规则：

- 原生 tool calling 足够稳定时，直接透传给 runtime
- 原生 structured output 不可靠时，适配层应配合 schema repair 或重试
- 原生 usage 不完整时，适配层应补充本地估算
- 原生 stop reason 不统一时，适配层应映射到标准终止原因

## 不应放在 Model Provider Adapter 的职责

以下职责不应下沉到模型适配层：

- 任务状态机
- 权限审批
- 子代理调度
- 上下文 compact 策略
- 长输出落盘
- 业务侧 prompt 策略
- sandbox 或 execution target 适配
- 凭据代理、vault 接入、tool proxy

这些属于 harness 或更上层模块。

## 规范结论

- `ModelProviderAdapter` 是多模型兼容的唯一差异吸收层
- 所有语言实现都应维持同一份标准能力视图
- 供应商专有字段不得泄漏到上层模块边界
- 执行环境差异不应被错误下沉到 ModelProviderAdapter
