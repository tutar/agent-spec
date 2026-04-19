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

## 标准接口

```text
ContextGovernance
  - analyze(messages, tools) -> context_report
  - should_compact(messages, model) -> boolean
  - compact(messages) -> compact_result
  - externalize_tool_result(result) -> processed_result
```

## 推荐策略

- 在接近阈值前触发 proactive compact
- 在 `prompt_too_long` 后触发 reactive compact
- 对超长工具输出做文件持久化，仅回传 preview
- 对当前 turn 维护单独的 token budget tracker
- 若底层模型原生支持 prompt cache、server-side truncation 或 context window 管理，agent 应优先接入这些能力，但仍保留统一预算与恢复语义

## 与 Session 的边界

- `Session`
  负责可恢复的原始事件历史
- `ContextGovernance`
  负责把这些历史塑形成当前模型上下文

规范上必须允许：

- session 保持完整
- harness 对送入模型的上下文做裁剪、重排、摘要或缓存优化

不能把“给模型看的上下文”误当成“系统保存的历史”。

## 不可缺失的治理对象

- assistant 历史消息
- tool call / tool result
- attachment 内容
- memory 注入内容
- system prompt 与 tool schema 的固定成本

## 为什么要落盘而不是截断

直接截断工具结果会损失可恢复性。外存化的收益是：

- 保留完整结果
- prompt 中只保留 preview
- 后续需要时可显式 read back
- 不破坏当前 turn 的上下文预算

## 当前仓库映射


## 规范结论

- 长会话治理必须系统化
- compact、budget、外存化应统一纳入同一模块
- 上下文治理需要和 runtime 深度集成，不能做成纯后处理
- 治理策略必须可跨模型迁移
- recoverable history 属于 session，context shaping 属于 harness
