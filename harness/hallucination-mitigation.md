# Hallucination Mitigation

## 职责

本文总结 agent runtime 如何系统性降低 hallucination 风险。

这里的 hallucination 不只指普通问答中的事实幻觉，还包括：

- action hallucination
  把“想做某动作”误当成“动作已经发生”
- tool hallucination
  在没有真实 schema、真实返回值或真实权限的前提下假定工具可用、工具已成功
- completion hallucination
  在没有独立证据时过早宣称任务完成
- evidence-loss hallucination
  在 compact、长结果裁剪、resume 后开始靠模型“回忆”过去的事实
- long-context drift
  长会话中连续性丢失后用似是而非的补全替代真实上下文

它是一份 `Harness` 下的建议性方案，而不是新的顶层模块或单一子系统。

## 核心结论

没有单一的 anti-hallucination prompt trick。

更稳定的做法是把多层机制组合起来：

```text
grounding
  + validation
  + controlled capability exposure
  + faithful reporting
  + context governance
  + independent verification
```

其中：

- `grounding`
  防止模型脱离证据回答
- `validation`
  防止错误 tool call、错误结构、错误输入继续推进
- `controlled capability exposure`
  防止模型在未知工具面上乱猜
- `faithful reporting`
  防止把未验证、失败或部分完成表述成成功
- `context governance`
  防止长会话漂移和证据丢失
- `independent verification`
  防止主 agent 自评式误判

## 现有机制可抽象出的防幻觉层

### 1. Independent Verification

独立 verifier 是防 completion hallucination 的最强一层。

建议语义：

- 主 agent 不应把自己的检查视为充分验证
- verifier 应使用独立角色、独立约束和独立 verdict
- verifier 应以真实命令输出和 adversarial probe 为证据，而不是代码阅读摘要

这层主要缓解：

- “看起来差不多完成了”的误判
- 只跑 happy path 就宣称成功
- 主 agent 自己给自己背书

相关规范见 [task/verification.md](task/verification.md)。

### 2. Validate Before Execute

tool invocation 在执行前应至少经过：

- input validation
- permission / policy gate
- tool-specific semantic checks

建议：

- 输入校验失败应尽早收敛为 validation / protocol error
- 不应因为工具名匹配就继续执行
- 对 destructive 或高风险工具，应要求更强的前置校验

这层主要缓解：

- tool argument hallucination
- “工具应该能跑”式误判
- 参数形状错了但后续推理继续把执行当成成功

相关接口见 [../tools/tool-model/tool-definition.md](../tools/tool-model/tool-definition.md)。

### 3. Fail Fast On Invalid Structure

对 structured output、tool result envelope 或其它机器消费结果，建议采用：

- schema validation
- retry cap
- 显式错误终态

而不是无限重试直到得到一个“看起来像成功”的结构。

建议：

- 非法结构必须被视为真实失败
- schema mismatch 应有清晰错误面
- retry exhaustion 后应 fail fast，而不是继续隐藏问题

这层主要缓解：

- structured output hallucination
- schema-shape drifting
- 依赖方误把无效结构当作可信结果

### 4. Controlled Capability Exposure

当工具面很大、很动态、或 schema 很重时，建议采用 deferred capability exposure：

- 先暴露名称或类别
- 真正调用前再拉取完整 schema
- 在 schema 未加载前不可调用

这层主要缓解：

- tool hallucination
- 模型对未知参数形状的乱猜
- 大工具面导致的 context 污染

建议：

- 对 workflow-specific、MCP-like、动态接入工具优先使用 discovery / defer
- 保留 `always-load` 小集合，给必须 turn-1 可见的关键能力

### 5. Preserve Evidence Instead Of Reconstructing It

对超长工具输出、外部读取得到的原始证据和后台任务产物，建议保留可恢复引用，而不是只保留口头摘要。

推荐做法：

- 大结果外存化，仅把 preview 放进当前上下文
- 保留 `persisted_ref` / `output_ref` / `evidence_ref`
- 后续需要时显式 read-back，而不是依赖模型记忆

这层主要缓解：

- evidence-loss hallucination
- compact 后工具结果失真
- summary 代替原始证据后发生再加工偏差

### 6. Context Drift Mitigation

长会话里的 hallucination 很多并不是“模型瞎编新事实”，而是：

- 忘了前面已经确认过什么
- 把旧上下文和新上下文拼错
- compact 后 continuity 丢失

因此建议把以下能力视为 anti-hallucination 组成部分：

- proactive compact
- reactive compact
- short-term continuity summary
- nested / relevant memory injection
- prompt cache break detection

这层主要缓解：

- long-context drift
- resume 后的补脑
- compact 后的语义漂移

相关规范见 [context-assembly/context-governance.md](context-assembly/context-governance.md)。

### 7. Faithful Reporting Guardrails

很多 hallucination 发生在“如何汇报结果”，而不是“如何得到结果”。

建议在 bootstrap/system guidance 中明确要求：

- 没验证过就不要声称完成
- 检查失败就如实报告失败
- 未执行的步骤要显式说未执行
- 不要把 caveat 包装成 green result

对 tool results 与 external sources，还建议加入 prompt-injection / evidence-corruption 提示，提醒模型不要把外部返回值自动当成可信指令。

### 8. Post-Turn Reflection Is Helpful But Secondary

stop hooks、memory extraction、dream/consolidation 这类机制更适合：

- 经验提炼
- continuity 修复
- 背景校正

它们可以减少后续 session 的漂移，但不应替代：

- 当前任务的 grounding
- 当前结果的 correctness verification

换句话说：

- `reflection` 有助于减少长期偏差
- `verification` 才是抑制当前任务伪完成的关键层

## 推荐的实现原则

### Evidence First

- 能引用真实工具结果就不要复述成无来源结论
- 能保留 evidence ref 就不要只保留摘要
- 能读取外部资源就不要靠内部推测补足

### Separate Continuity From Truth

- continuity summary 只服务继续工作
- memory / compact summary 不是事实源
- 真正的事实锚点应来自 transcript、resource read、tool result 或 verification evidence

### Prefer Narrow Trusted Paths

- 对高风险回答优先走 read-only、schema-validated、permission-gated 路径
- 对未知能力优先延迟暴露
- 对高风险动作要求更高等级的 explicit authorization

### Verify Non-Trivial Completion

- 非 trivial 实现不应仅凭主 agent 自查就结束
- 应进入独立 verification gate
- verdict 应来自 verifier，而不是实现者

### Keep Failure Modes Explicit

- validation failure、permission deny、structured output mismatch、verification fail 都应显式表达
- 不要把这些失败收敛成一个模糊的“模型不太稳定”

## 建议的扩展对象

这几项作为推荐扩展即可，不必视为核心强制对象：

```text
VerificationPolicy
  - should_verify(turn_or_task) -> boolean
  - verifier_profile
```

```text
GroundingPolicy
  - requires_evidence(response_type) -> boolean
  - allowed_evidence_kinds[]
```

```text
EvidenceRef
  - kind: tool_result | task_output | resource_read | transcript_span
  - ref
  - preview?
```

```text
StructuredOutputGuard
  - schema
  - retry_limit
  - on_exhaustion
```

```text
CapabilityExposurePolicy
  - deferred_capabilities[]
  - always_loaded_capabilities[]
  - discovery_path
```

## 边界与误区

- anti-hallucination 不等于“默认联网查一切”
- permission 主要防 action hallucination，不直接保证 factual correctness
- compact / memory 主要防 continuity drift，不直接提供 truth guarantee
- observability 有助于发现 hallucination，但不是一线防护
- verifier 是 correctness gate，不应退化成“再让同一个 agent 想一遍”

## 规范结论

- 反幻觉更适合作为 `Harness` 的组合式建议方案，而不是单一子系统
- 最值得优先沉淀的是：grounding、validation、evidence preservation、context governance、independent verification
- 对 production agent 而言，“faithful reporting” 与 “independent verification” 往往比单纯 prompt 优化更关键
