# Bootstrap Prompts

## 职责

`BootstrapPrompts` 定义 harness 在进入主 query loop 前必须建立的 `system prompt skeleton`。

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

## 要解决的问题

如果没有独立的 bootstrap prompt 层，系统通常会退化成：

- 把所有 context 混拼成一个大字符串
- 无法对 system prompt 做 section 级缓存
- custom prompt 与默认 prompt 语义冲突
- prompt cache 命中率不稳定
- 模型行为骨架和动态上下文装配互相污染

因此规范上必须把 `BootstrapPrompts` 从普通 context provider 中独立出来。

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

PromptBlocks
  - attribution_prefix?
  - static_blocks[]
  - dynamic_blocks[]
```

## 规范要求

### 1. Bootstrap prompt 必须是分段对象

实现不应将 bootstrap prompt 直接建模成单一字符串。

原因：

- 不利于按 section 做缓存
- 不利于做 static / dynamic 边界切分
- 不利于 override 和 append 的稳定组合

### 2. 必须支持优先级覆盖

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

bootstrap prompt 不是 `systemContext` 或 `userContext` 的别名。

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

触发失效的典型原因包括：

- clear
- compact
- worktree / cwd 切换
- integration capability 变化
- late-bound server instruction 变化

### 5. 必须支持 static / dynamic boundary

bootstrap prompt 应支持将：

- 稳定前缀
- 会话动态部分

显式分开。

这个边界不只是优化项，而是 prompt caching 和 API transport block projection 的稳定输入。

### 6. 动态 section 必须是显式声明

下列内容通常属于动态 section：

- session guidance
- memory mechanics prompt
- environment info
- output style
- MCP server instructions
- token budget instructions

实现不应把这些动态段偷偷混入静态前缀中。

## 推荐默认策略

- 默认 prompt 由静态 section 和动态 section 两组组成
- 静态 section 尽量稳定，优先进入 cacheable prefix
- 动态 section 延后解析，并支持独立失效
- append prompt 永远在最终尾部
- custom prompt 应是完整替换，除非调用方明确要求 layered merge

## 与其它模块的边界

- 与 [context-provider.md](context-provider.md)
  `BootstrapPrompts` 负责 system prompt skeleton；`ContextProvider` 负责 structured context 和 attachment assembly
- 与 [prompt-cache-strategy.md](prompt-cache-strategy.md)
  `BootstrapPrompts` 定义 prompt skeleton 的结构；`PromptCacheStrategy` 定义这些结构如何组织成 cache-friendly 输入
- 与 [context-governance.md](context-governance.md)
  `BootstrapPrompts` 提供可分块输入；`ContextGovernance` 负责预算、compact 和大上下文治理
- 与 [../model-provider/model-provider-adapter.md](../model-provider/model-provider-adapter.md)
  `ModelProviderAdapter` 决定如何把 prompt blocks 投影到特定模型协议；不定义 bootstrap prompt 内容

## 默认实现映射

当前仓库中的默认实现映射为：

- 默认 prompt section 注册见 [constants/prompts.ts](../../../cc/constants/prompts.ts)
- section 缓存与解析见 [constants/systemPromptSections.ts](../../../cc/constants/systemPromptSections.ts)
- 优先级合成见 [utils/systemPrompt.ts](../../../cc/utils/systemPrompt.ts)
- query 前三段装配见 [utils/queryContext.ts](../../../cc/utils/queryContext.ts)
- systemContext 追加见 [query.ts](../../../cc/query.ts) 和 [utils/api.ts](../../../cc/utils/api.ts)
- prompt block 切分与 cache scope 投影见 [utils/api.ts](../../../cc/utils/api.ts) 和 [services/api/claude.ts](../../../cc/services/api/claude.ts)

## 规范结论

- bootstrap prompt 应是 harness 中独立的一层
- bootstrap prompt 应以 section 为基本单元，而不是单一字符串
- bootstrap prompt 应支持优先级覆盖、section 级缓存与边界切分
- bootstrap prompt 应与 structured context、attachment context、memory recall 分开建模
