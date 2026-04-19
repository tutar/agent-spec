# Context Governance

## 职责

`ContextGovernance` 负责控制长会话中的上下文膨胀和预算退化。

它不是优化项，而是 production agent runtime 的必备模块。

它也承担一部分 anti-hallucination 职责：通过 compact、外存化和 continuity shaping，降低长会话中的 context drift 与 evidence loss。

它负责的是 harness 内部的 context shaping，不等于 durable session storage。

## 必须覆盖的能力

- token usage 估算
- 阈值预警
- auto-compact
- overflow recovery
- 长 tool result 外存化
- budget-driven continuation
- delta injection
- recall budget control
- continuity shaping

## 标准接口

```text
ContextGovernance
  - analyze(context_assembly_input, model_profile) -> context_report
  - should_compact(context_report, runtime_state) -> boolean
  - compact(transcript_window, runtime_state) -> compact_result
  - externalize(payload, policy) -> processed_payload
  - plan_delta(previous_visible_state, current_state) -> delta_plan
```

## 核心治理对象

- assistant 历史消息
- tool call / tool result
- attachment 内容
- memory 注入内容
- bootstrap prompt 与 tool schema 的固定成本
- dynamic recall / delta 的边际成本

## 设计要求

### 1. compact summary 不是 truth source

compact 的职责是 continuity shaping，不是替代 transcript、tool result 或持久化 evidence。

### 2. 大结果应优先外部化

当工具结果、诊断输出或资源读取结果过长时，推荐：

- 在当前轮只保留 preview
- 将完整内容外部化
- 通过 `EvidenceRef` 或等价引用在后续显式 read back

### 3. delta 注入优先于重写静态 prompt

频繁变化的信息，例如：

- capability changes
- instruction changes
- date change
- runtime notices

优先以增量方式进入 message/attachment 平面，而不是反复改写静态 prompt 前缀。

### 4. recall 必须有独立预算

memory recall、relevant evidence surfacing 与普通 attachment 不能共用无界预算。

### 5. governance 必须与 runtime 深度集成

它不能只作为 turn 结束后的后处理。

它必须参与：

- 本轮上下文分析
- 是否 compact 的决策
- long result externalization
- prompt-too-long 后的恢复

## 与 Session 的边界

- `Session`
  负责可恢复的原始事件历史
- `ContextGovernance`
  负责把这些历史塑形成当前模型上下文

规范上必须允许：

- session 保持完整
- harness 对送入模型的上下文做裁剪、重排、摘要或缓存优化

不能把“给模型看的上下文”误当成“系统保存的历史”。

## 推荐策略

- 在接近阈值前触发 proactive compact
- 在 `prompt_too_long` 或语义等价错误后触发 reactive compact
- 对超长工具输出做外部化，仅回传 preview
- 对 recall、attachments、tool schemas 维护单独 budget 视图
- 将动态变化面优先投影成 delta，而不是重建静态前缀
- 若底层模型原生支持 prompt cache、server-side truncation 或 context window 管理，agent 应优先接入这些能力，但仍保留统一预算与恢复语义

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

- token 分析、compact 决策、外部化通常由本地 harness 基于本地 transcript slice 和本地结果存储完成

### Cloud-Compatible Mapping

- 治理逻辑仍在 harness 内部
- transcript slice、tool result、memory recall 可以来自远程接口
- 外部化目标可以是远程对象存储或远程 result store

## 规范结论

- 长会话治理必须系统化
- compact、budget、外存化应统一纳入同一模块
- 上下文治理需要和 runtime 深度集成，不能做成纯后处理
- 治理策略必须可跨模型迁移
- recoverable history 属于 session，context shaping 属于 harness
