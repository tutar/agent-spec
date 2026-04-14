# Prompt Cache Strategy

## 职责

`PromptCacheStrategy` 定义 harness 如何为了更好利用底层模型或 provider 的 prompt cache 能力而组织输入。

它关注的是：

- 如何构造稳定前缀
- 如何放置 cache marker / cache breakpoint
- 如何避免不必要的 cache bust
- 如何在 fork / subagent / side-query 中共享缓存命中条件
- 如何对 cache read / write / break 做一致性观测

它不负责：

- 底层模型的 KV cache 实现
- durable session storage
- context governance 的预算策略本身
- provider client 的认证与传输

## 要解决的问题

即使底层 provider 原生支持 prompt caching，如果 harness 不主动稳定 prompt 结构，也会持续丢失缓存命中。

典型失败模式包括：

- system prompt 每轮字节级变化
- 动态上下文混入静态前缀
- TTL / scope 在会话中途切换
- fork agent 改变 cache-critical 参数
- tool result 回填方式不稳定

因此，prompt caching 不应只被视为 provider 功能，而应被视为 harness 的上下文装配策略之一。

## 稳定接口

推荐最小接口：

```text
PromptCacheStrategy
  - prepare_cacheable_prefix(prompt_layers, context_plan) -> CacheablePromptPlan
  - place_cache_markers(prompt_plan, message_plan) -> CacheAnnotatedInput
  - build_cache_policy(query_source, host_profile) -> CachePolicy
  - share_cache_with_fork(parent_turn, fork_options) -> ForkCachePlan
  - detect_cache_break(previous_call, current_call, usage) -> CacheBreakReport?
```

推荐最小对象：

```text
CachePolicy
  - strategy_id
  - enabled
  - cache_ttl
  - cache_scope
  - supports_prefix_reuse
  - supports_message_level_markers
  - supports_tool_result_reference

CacheablePromptPlan
  - stable_prefix
  - dynamic_suffix
  - cache_breakpoints[]
  - cache_sensitive_fields[]

ForkCachePlan
  - share_parent_prefix
  - inherited_cache_critical_params[]
  - skip_cache_write?
```

## 规范要求

### 1. Harness 必须是 prompt-cache-aware

若底层 provider 支持 prompt caching 或等价能力，harness 不应仅“透传开关”，而应主动：

- 稳定前缀
- 分离静态与动态内容
- 控制 cache marker 位置
- 避免无意义的 cache bust

### 2. Prompt cache 策略必须与 provider 语义解耦

规范不应把某个 provider 的私有字段直接写成通用标准。

例如：

- Anthropic 的 `cache_control`
- Anthropic 的 `cache_reference`

这些都只能出现在默认实现映射中，而不应成为跨语言、跨 provider 的稳定接口字段名。

### 3. 必须显式建模静态前缀与动态后缀

实现至少应能区分：

- cacheable stable prefix
- session / turn dynamic suffix

否则 provider 即使支持 prefix cache，也难以稳定命中。

### 4. 必须支持 cache-critical 参数稳定

实现应显式跟踪哪些参数会影响 cache 命中。

典型包括：

- model identity
- prompt section bytes
- cache TTL / scope
- tool schema surface
- thinking / effort mode
- forked subagent 继承参数

### 5. 必须支持 cache break detection

若 provider 返回 cache read / write 使用量或等价指标，SDK 应允许：

- 检测异常 cache miss
- 判断是预期变化还是异常 break
- 将 break 作为可观测事件暴露给外部

### 6. 必须允许 host-neutral 实现

`TUI / Desktop / Cloud` 三种 host 都应共享同一套 prompt cache 语义。

允许不同 host：

- 使用不同 transport
- 使用不同 session store
- 使用不同 sandbox

但不允许因此改变：

- stable prefix 的定义
- cache break 的语义
- fork cache sharing 的语义

## 推荐策略分层

### 1. Anthropic Native Strategy

用于底层 provider 原生提供 Anthropic-style prompt caching 语义时。

特征通常包括：

- prefix-level cache marker
- request block ordering 敏感
- message / tool result 也可参与缓存
- provider 返回 cache read / write usage

当前仓库默认实现属于这一类。

### 2. OpenClaw-Mediated Strategy

用于底层 provider 能力不统一，但系统通过 OpenClaw 一类中间层统一了 prompt cache 语义。

特征通常包括：

- provider-native cache 能力被上提为统一策略接口
- harness 面向统一 cache policy，而不是直接写 provider 私有参数
- 可覆盖 OpenAI / Anthropic / Gemini 等不同 provider

### 3. Fallback Strategy

用于底层 provider 没有原生 prompt cache 或能力不足时。

这时 SDK 仍应尽量做：

- stable prefix construction
- dynamic suffix isolation
- fork parameter stability
- cache-break-sensitive context assembly

即使没有真正的 server-side cache，也能减少无效重组与未来迁移成本。

## 与其它模块的边界

- 与 [bootstrap-prompts.md](bootstrap-prompts.md)
  `BootstrapPrompts` 定义 system prompt skeleton；`PromptCacheStrategy` 定义如何让这些 skeleton 更适合被缓存复用
- 与 [context-provider.md](context-provider.md)
  `ContextProvider` 决定上下文来源；`PromptCacheStrategy` 决定这些上下文如何组织成 cache-friendly 输入
- 与 [context-governance.md](context-governance.md)
  `ContextGovernance` 处理预算、compact、overflow；`PromptCacheStrategy` 处理 cache reuse 与 cache stability
- 与 [../model-provider/model-capability-routing.md](../model-provider/model-capability-routing.md)
  模型能力路由决定是否启用 provider-native prompt caching；`PromptCacheStrategy` 负责实际组织输入

## 默认实现映射

当前仓库中的默认实现映射为：

- system prompt section cache 见 [../../../constants/systemPromptSections.ts](../../../cc/constants/systemPromptSections.ts)
- static / dynamic boundary 见 [../../../constants/prompts.ts](../../../cc/constants/prompts.ts)
- system prompt block 切分见 [../../../utils/api.ts](../../../cc/utils/api.ts)
- provider-native cache marker / cache reference 投影见 [../../../services/api/claude.ts](../../../cc/services/api/claude.ts)
- cache break detection 见 [../../../services/api/promptCacheBreakDetection.ts](../../../cc/services/api/promptCacheBreakDetection.ts)
- fork cache sharing 见 [../../../utils/forkedAgent.ts](../../../cc/utils/forkedAgent.ts) 与 [../../../tools/AgentTool/forkSubagent.ts](../../../cc/tools/AgentTool/forkSubagent.ts)

当前默认实现应视为：

- `AnthropicNativePromptCacheStrategy`

而不是跨 provider 的唯一标准。

## 规范结论

- prompt caching 不是单纯 provider 开关，而是 harness 的输入组织策略
- 当前源码应沉淀为 `Anthropic Native Strategy`
- 其他 provider 可以通过 `OpenClaw-Mediated Strategy` 接入统一语义
- 规范应稳定策略接口，不应稳定某个 provider 的私有字段名
