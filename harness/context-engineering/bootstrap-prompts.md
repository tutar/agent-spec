# Bootstrap Prompts

## 职责

`BootstrapPrompts` 定义 `Harness.ContextEngineering` 在进入主 query loop 前必须建立的 `system prompt skeleton`。

它负责：

- 为 agent 提供稳定的初始行为约束
- 提供模型可见的身份、能力和环境说明
- 以分段方式组织系统提示词
- 为 prompt caching 提供 static / dynamic boundary
- 为 override / agent / custom / append 等上层调用提供稳定合成规则

它不负责：

- durable session transcript
- attachment context
- tool result 注入
- memory recall 结果本身

## 稳定接口

推荐最小接口：

```text
BootstrapPromptAssembler
  - build_default_prompt(runtime_capabilities, model_view) -> PromptSections
  - merge_prompt_layers(base, overrides) -> PromptSections
  - resolve_sections(sections, runtime_state) -> ResolvedPromptSections
  - split_static_dynamic(sections) -> PromptBlocks
  - invalidate_sections(reason) -> void
```

推荐最小对象：

```text
PromptSection
  - name
  - text | compute()
  - cache_policy
  - cache_breaking
  - dynamic
```

```text
PromptBlocks
  - attribution_prefix?
  - static_blocks[]
  - dynamic_blocks[]
```

## 规范要求

### 1. bootstrap prompt 必须是分段对象

实现不应将 bootstrap prompt 直接建模成单一字符串。

### 2. 必须支持覆盖与追加语义

至少应支持这些语义层：

- override prompt
- coordinator / orchestrator prompt
- agent prompt
- custom prompt
- default prompt
- append prompt

实现必须明确：

- 哪些层是替换
- 哪些层是追加
- 哪些层能与 default 共存

### 3. 必须与 structured context 分离

bootstrap prompt 不是 `system_context` 或 `user_context` 的别名。

推荐分层：

- bootstrap prompt
  system prompt 骨架
- structured context
  结构化 system/user/session context
- attachment context
  以 message 方式进入模型的动态上下文

### 4. 必须支持 section 级缓存与失效

实现应允许每个 prompt section 单独声明：

- 可缓存
- 不可缓存
- 何时失效
- 是否会破坏 prompt cache

### 5. 必须支持 static / dynamic boundary

bootstrap prompt 应支持将：

- 稳定前缀
- 会话动态部分

显式分开。

这个边界不只是优化项，而是 prompt caching 和 API projection 的稳定输入。

### 6. 动态 section 必须显式声明

下列内容通常属于动态 section：

- session guidance
- memory mechanics prompt
- environment info
- output style
- server / integration instructions
- token budget instructions

实现不应把这些动态段偷偷混入静态前缀中。

### 7. 必须区分 agent prompt 与 startup-only briefing

`agent prompt` 是 bootstrap prompt 内的稳定语义层，用来表达：

- identity / role
- operating mode
- domain-specific behavior contract
- 与 default prompt 的覆盖或叠加关系

首轮 kickoff、session-start、agent-start、resume-start 这类内容应走独立 startup context，见 [startup-and-turn-zero-context.md](startup-and-turn-zero-context.md)。

## 推荐默认策略

- 默认 prompt 由静态 section 和动态 section 两组组成
- 静态 section 尽量稳定，优先进入 cacheable prefix
- 动态 section 延后解析，并支持独立失效
- append prompt 永远在最终尾部
- custom prompt 应是完整替换，除非调用方明确要求 layered merge
- agent prompt 视为稳定 prompt layer；首轮额外引导默认走 startup context
- environment summary 可在 bootstrap prompt 中投影为 section，但其归属应先由 structured context 层定义

## 与其它模块的边界

- 与 [context-input-model.md](context-input-model.md)
  本页只定义 system prompt skeleton；输入平面见 `ContextInputModel`
- 与 [context-provider.md](context-provider.md)
  `BootstrapPrompts` 负责 system prompt skeleton；`ContextProvider` 负责向不同 context plane 提供结构化片段
- 与 [context-assembly-pipeline.md](context-assembly-pipeline.md)
  `BootstrapPrompts` 只定义装配前半段的 system skeleton；完整顺序见 `ContextAssemblyPipeline`
- 与 [prompt-cache-strategy.md](prompt-cache-strategy.md)
  `BootstrapPrompts` 定义 prompt skeleton 的结构；`PromptCacheStrategy` 定义这些结构如何组织成 cache-friendly 输入
- 与 [context-governance.md](context-governance.md)
  `BootstrapPrompts` 提供可分块输入；`ContextGovernance` 负责预算、compact 和大上下文治理
- 与 [../model-provider/model-provider-adapter.md](../model-provider/model-provider-adapter.md)
  `ModelProviderAdapter` 决定如何把 prompt blocks 投影到特定模型协议；不定义 bootstrap prompt 内容

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

- section registry 常由本地常量、配置和 runtime state 解析得到
- section cache 常与本地 harness 生命周期协同

### Cloud-Compatible Mapping

- bootstrap prompt 仍在 harness 内部构造
- runtime state、integration capability、environment context 可来自远程模块接口
- section 语义与 static/dynamic boundary 不因部署位置改变

## 规范结论

- bootstrap prompt 应是 harness 中独立的一层
- bootstrap prompt 应以 section 为基本单元，而不是单一字符串
- bootstrap prompt 应支持优先级覆盖、section 级缓存与边界切分
- bootstrap prompt 应与 structured context、attachment context、memory recall 分开建模
- agent prompt 应被建模为稳定语义层，而不是首轮一次性引导容器
