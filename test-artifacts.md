# Test Artifacts

## 目标

本文件定义跨语言共享的测试工件格式。

这些工件用于验证不同 agent 实现是否保持一致行为。

## 工件类型

### 1. Event Recording

用于记录一段标准 runtime 事件流。

推荐字段：

```text
EventRecording
  - artifact_id
  - schema_version
  - scenario
  - events
```

### 2. Replay Fixture

用于重放固定输入并验证输出行为。

```text
ReplayFixture
  - artifact_id
  - schema_version
  - input
  - expected_outputs
  - expected_terminal_state
```

### 3. Compatibility Fixture

用于验证某个能力面在不同模型或不同语言实现中的一致性。

```text
CompatibilityFixture
  - artifact_id
  - target_module
  - preconditions
  - assertions
```

### 4. Object Fixture

用于验证 canonical object 的最小字段、可序列化性和兼容语义。

```text
ObjectFixture
  - artifact_id
  - schema_version
  - object_name
  - payload
  - invariants
```

## 推荐场景

- tool execution
- requires_action
- compact boundary
- session wake / resume
- memory recall injection
- `AGENTS.md` memory injection precedence
- background task completion
- remote task status sync
- policy decision serialization
- persisted tool result resume
- memory consolidation background safety
- skills discovery precedence
- skills activation wrapping
- skills context protection

当前规范目录下的第一批工件样本位于：

- [conformance/golden/basic-turn.events.json](conformance/golden/basic-turn.events.json)
- [conformance/golden/tool-call-roundtrip.events.json](conformance/golden/tool-call-roundtrip.events.json)
- [conformance/golden/requires-action-approval.events.json](conformance/golden/requires-action-approval.events.json)
- [conformance/golden/session-resume.event-log.json](conformance/golden/session-resume.event-log.json)
- [conformance/golden/sandbox-deny.events.json](conformance/golden/sandbox-deny.events.json)
- [conformance/golden/memory-recall-and-consolidation.json](conformance/golden/memory-recall-and-consolidation.json)
- [conformance/golden/agents-memory-loading-precedence.json](conformance/golden/agents-memory-loading-precedence.json)
- [conformance/golden/policy-ask-deny-allow.json](conformance/golden/policy-ask-deny-allow.json)
- [conformance/golden/persisted-tool-result-resume.json](conformance/golden/persisted-tool-result-resume.json)
- [conformance/golden/memory-consolidation-background-safety.json](conformance/golden/memory-consolidation-background-safety.json)
- [conformance/golden/skills-discovery-precedence.json](conformance/golden/skills-discovery-precedence.json)
- [conformance/golden/skills-activation-wrapping.json](conformance/golden/skills-activation-wrapping.json)
- [conformance/golden/skills-context-protection.json](conformance/golden/skills-context-protection.json)
- [conformance/golden/cloud-wake-and-reprovision.json](conformance/golden/cloud-wake-and-reprovision.json)
- [conformance/golden/prompt-cache-stable-prefix.json](conformance/golden/prompt-cache-stable-prefix.json)
- [conformance/golden/prompt-cache-dynamic-suffix.json](conformance/golden/prompt-cache-dynamic-suffix.json)
- [conformance/golden/prompt-cache-fork-sharing.json](conformance/golden/prompt-cache-fork-sharing.json)
- [conformance/golden/prompt-cache-break-detection.json](conformance/golden/prompt-cache-break-detection.json)
- [conformance/golden/prompt-cache-strategy-equivalence.json](conformance/golden/prompt-cache-strategy-equivalence.json)

## 规范结论

- 测试工件应脱离具体语言实现
- 所有语言 agent 实现 应共享同一套录制样本和重放样本语义
- canonical object fixture 应优先覆盖 `PolicyDecision`、`ResumeSnapshot`、`DurableMemoryRecord`、`PersistedToolResultRef`
- memory injection object fixture 应优先覆盖 `DurableMemoryInjectionSource`、`LoadedMemoryInjection`
- skills 相关 object fixture 应优先覆盖 `ImportedSkillManifest`、`SkillCatalogEntry`、`SkillActivationResult`
