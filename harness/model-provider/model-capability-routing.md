# Model Capability Routing

## 目标

Agent SDK 不应与某个模型厂商、某种协议或某种语言绑定。

它的核心职责是充当 `LLM Harness`：

- 屏蔽不同模型在接口与能力上的差异
- 统一对外暴露 agent runtime 语义
- 在模型原生能力不足时由 SDK 补齐
- 在模型原生能力已经足够时直接使用模型能力

## 核心原则

- `Native First`
  模型已经稳定提供的能力，应优先直接使用。
- `Harness Fallback`
  模型不提供、提供不稳定、提供但不可移植的能力，应由 SDK 落地。
- `Behavioral Consistency`
  即使底层由模型原生提供，上层也应看到一致的 runtime 语义。
- `Capability Negotiation`
  SDK 应在运行时根据模型能力决定采用原生路径还是 harness 路径。

## 典型能力分类

- 模型原生可直接承接的能力
  例如 structured output、tool calling、reasoning token、JSON schema 约束、原生 web search。
- 需由 SDK 补齐的能力
  例如任务状态机、权限系统、上下文治理、跨工具并发编排、requires_action、长输出外存化。
- 混合能力
  例如函数调用、内置搜索、文件读写、代码执行。模型可能提供调用接口，但 SDK 仍需负责策略控制和运行时一致性。

## 能力路由要求

SDK 在每次会话或每轮 turn 开始前，应能得到一个模型能力视图。

建议至少包含：

- 是否支持原生 tool calling
- 是否支持并行 tool calling
- 是否支持结构化输出约束
- 是否支持流式 delta
- 是否支持 server-side search/fetch
- 是否支持 reasoning budget 或等价控制
- 是否支持 prompt caching 或等价机制

## 路由决策规则

1. 对模型原生能力做探测或配置声明
2. 评估该能力是否满足稳定性、一致性、可观测性要求
3. 满足要求则直接使用原生能力
4. 不满足要求则切换到 SDK harness 实现
5. 对上层暴露统一语义，不暴露底层分歧

## 示例

- tool calling
  若模型原生 tool calling 已稳定，SDK 直接使用该协议；若模型不支持，则 SDK 可退化为 prompt-level action planning。
- structured output
  若模型能稳定遵守 schema，则直用；若经常偏离，则 SDK 增加校验、重试或 repair。
- web search
  若模型具备可信原生搜索，可直接路由；若不具备，则由 SDK 挂接外部搜索工具。
- prompt caching
  若 provider 原生支持 prompt caching，则 SDK 应启用对应 `PromptCacheStrategy`；若 provider 语义不统一，则可通过 OpenClaw 一类中间层统一后再接入 harness。

provider 能力探测与 provider-native 协议映射应通过
`ModelProviderAdapter` 进入 routing，而不是直接散落在 runtime 分支中。

## 不应由模型承担的能力

以下能力即使模型很强，通常仍应由 SDK 负责：

- 权限与审批
- 任务生命周期管理
- 会话状态同步
- 审计与可观测性
- 长上下文治理
- 多子代理调度
- prompt cache 的前缀稳定化与 cache break 检测

## 规范结论

- SDK 的定位是 harness，不是模型包装层
- 能力路由必须是运行时能力，而不是编译时写死
- 所有语言实现都应共享同一套能力路由语义
- provider-native prompt caching 与 OpenClaw-mediated prompt caching 都应被视为能力路由输入，而不是硬编码分支
