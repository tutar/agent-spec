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
```

```text
CacheablePromptPlan
  - stable_prefix
  - dynamic_suffix
  - cache_breakpoints[]
  - cache_sensitive_fields[]
```

```text
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

### 3. 必须显式建模静态前缀与动态后缀

实现至少应能区分：

- cacheable stable prefix
- session / turn dynamic suffix

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

若 provider 返回 cache read / write 使用量或等价指标，agent 应允许：

- 检测异常 cache miss
- 判断是预期变化还是异常 break
- 将 break 作为可观测事件暴露给外部

### 6. 必须允许 host-neutral 实现

`Local / Cloud` 两种 host 都应共享同一套 prompt cache 语义。

## 推荐策略分层

### 1. Anthropic Native Strategy

用于底层 provider 原生提供 Anthropic-style prompt caching 语义时。

### 2. OpenClaw-Mediated Strategy

用于底层 provider 能力不统一，但系统通过中间层统一了 prompt cache 语义。

### 3. Fallback Strategy

用于底层 provider 没有原生 prompt cache 或能力不足时。

这时 agent 仍应尽量做：

- stable prefix construction
- dynamic suffix isolation
- fork parameter stability
- cache-break-sensitive context assembly

## 与其它模块的边界

- 与 [bootstrap-prompts.md](bootstrap-prompts.md)
  `BootstrapPrompts` 定义 system prompt skeleton；`PromptCacheStrategy` 定义如何让这些 skeleton 更适合被缓存复用
- 与 [context-input-model.md](context-input-model.md)
  本页只关心 cache-friendly 组织，不重新定义输入对象模型
- 与 [context-assembly-pipeline.md](context-assembly-pipeline.md)
  cache marker placement 必须服从 assembly pipeline，而不是替代它
- 与 [context-provider.md](context-provider.md)
  `ContextProvider` 决定上下文来源；`PromptCacheStrategy` 决定这些上下文如何组织成 cache-friendly 输入
- 与 [context-governance.md](context-governance.md)
  `ContextGovernance` 处理预算、compact、overflow；`PromptCacheStrategy` 处理 cache reuse 与 cache stability
- 与 [../model-provider/model-capability-routing.md](../model-provider/model-capability-routing.md)
  模型能力路由决定是否启用 provider-native prompt caching；`PromptCacheStrategy` 负责实际组织输入

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

- 本地 harness 常直接根据 section registry、tool schemas、message plan 放置 cache marker

### Cloud-Compatible Mapping

- prompt cache strategy 仍在 harness 内部执行
- 云端仅改变 provider transport 和缓存宿主位置，不改变 stable prefix、dynamic suffix、cache break 的语义

## 规范结论

- prompt caching 不是单纯 provider 开关，而是 harness 的输入组织策略
- 当前源码应沉淀为 `Anthropic Native Strategy`
- 其他 provider 可以通过 `OpenClaw-Mediated Strategy` 接入统一语义
- 规范应稳定策略接口，不应稳定某个 provider 的私有字段名
