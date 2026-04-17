# Context Provider

## 职责

`ContextProvider` 负责把不同来源的上下文装配进 runtime。

它不应被理解成“返回若干字符串的 provider”，而应被理解成 context assembly 的稳定接口。

源码表明，模型可见上下文至少来自三个不同注入平面：

- system prompt composition
- structured system/user context
- attachment message context

memory / instructions / dynamic discovery 可以落到上述不同平面中。

## 推荐分层

- `system`
  运行环境、平台规则、仓库状态、日期等稳定上下文。
- `system-prompt`
  作为 system prompt 骨架参与组装的高优先级指令层。
- `session`
  当前对话轮次内需要共享的临时状态或投影状态。
- `memory`
  长期记忆、项目知识、CLAUDE.md 一类静态知识。
- `attachments`
  文件、图片、外部资源等一次性附加上下文。
- `dynamic`
  只有在当前任务触发时才加载的上下文。

## 标准接口

```text
ContextChunk
  - name
  - content
  - cacheable
  - priority
  - token_cost
  - source: system-prompt | system | session | memory | attachment | dynamic

ContextProvider
  - compose_system_prompt(base_prompt, runtime_state)
  - get_system_context()
  - get_user_context()
  - get_attachment_context(runtime_input, message_state)
  - get_dynamic_context(runtime_input)
```

推荐补充：

```text
AttachmentAssemblyPlan
  - user_input_attachments
  - thread_shared_attachments
  - main_thread_attachments
  - ordering
```

推荐再补：

```text
MemoryRecallPlan
  - prefetch(query, runtime_scope)
  - collect(message_history, read_state)
  - session_budget
  - dedup_policy

DeltaContextPlan
  - compute_delta(current_state, announced_state)
  - emit_delta_attachment()
```

## 设计要求

- 每个上下文源都应独立缓存，不要做全量缓存
- 稳定上下文和高波动上下文必须分开
- 必须允许过滤或裁剪单个上下文块
- 必须能配合 token budget 做上下文构成分析
- system prompt 与 attachment context 必须分层
- memory 不应被强制限定为某一种注入平面
- attachment context 应被视为一等上下文来源，而不是附加实现细节
- 上下文来源应尽量保留 provenance，便于后续 dedup、cache、compact 和审计
- attachment context 应允许按 runtime scope 做差异化装配
- attachment assembly 应允许稳定顺序，而不是无序集合
- memory recall 应允许独立的 session budget 与 prefetch/collect 生命周期
- discovery、listing、delta、recall 这几种动态上下文语义应被区分

## 默认策略

- system prompt composition 单独处理 agent / coordinator / custom prompt 叠加
- Git 状态一类信息放在 `system`
- `CLAUDE.md` / rules / instructions 默认落到 `memory`
- 文件、资源、delta、hook 输出等上下文放在 `attachments`
- 只在命中特定条件时加载的 skills / memories / instructions 放在 `dynamic`
- 当底层模型支持原生 server-side attachment 或 cache 时，优先使用模型能力；语义不足的部分再由 agent 补齐
- 若底层 provider 支持 prompt caching 或等价机制，应与 `PromptCacheStrategy` 协同保证 stable prefix，而不是只透传 provider 开关
- 主线程与子代理可拥有不同的 attachment 装配面
- 高频变化能力信息优先走 delta attachment，而不是重写 system prompt
- relevant memory 应走独立 recall 管线，而不是普通附件拼接
- skill discovery、skill listing 与 capability delta 应按不同目标建模

## 不推荐做法

- 所有上下文提前拼成一个大字符串
- 把 session state 和 memory 混为一谈
- 把 system prompt 和 user/attachment context 混为一谈
- 每轮都重新扫描全部知识文件
- 不记录上下文来源，导致后续无法做预算和 compact
- 把 attachment context 当成无序集合处理
- 把 recall、listing、delta、discovery 视为同一种动态上下文

## 当前仓库映射

- [prompt-cache-strategy.md](prompt-cache-strategy.md) 负责把这些上下文进一步组织成 cache-friendly 输入

## 规范结论

- 上下文必须分层
- 上下文必须结构化
- 上下文必须可缓存
- 上下文必须允许多注入平面并存
- attachment context 必须被视为一等上下文
- attachment context 必须允许有序装配与 scope-aware 装配
- dynamic context 必须允许 recall / discovery / listing / delta 分层
- 上下文必须可被治理系统单独分析和裁剪
- 上下文接口必须语言无关
