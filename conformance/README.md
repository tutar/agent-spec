# Conformance Cases

## 目标

本目录提供跨语言、跨宿主共享的合规测试资产。

这些资产不绑定具体语言实现，也不绑定单一宿主形态，而是用于验证：

- 五个顶层模块的外部行为是否与 `agent-spec/` 当前正文一致
- `Local-first` 默认实现与 `Cloud-compatible` 边界是否保持语义稳定
- 核心能力与标准扩展是否按声明分别通过对应验收

## 目录结构

### Core Cases

- [cases/basic-turn.md](cases/basic-turn.md)
- [cases/tool-call-roundtrip.md](cases/tool-call-roundtrip.md)
- [cases/requires-action-approval.md](cases/requires-action-approval.md)
- [cases/session-resume.md](cases/session-resume.md)

### Harness.Permission

- [cases/policy-ask-deny-allow.md](cases/policy-ask-deny-allow.md)
- [cases/permission-rule-precedence-and-shadowing.md](cases/permission-rule-precedence-and-shadowing.md)
- [cases/permission-mode-and-headless-degradation.md](cases/permission-mode-and-headless-degradation.md)
- [cases/permission-update-and-scope.md](cases/permission-update-and-scope.md)

### Harness.Task

- [cases/background-agent.md](cases/background-agent.md)
- [cases/runtime-task-lifecycle.md](cases/runtime-task-lifecycle.md)
- [cases/task-output-cursor-and-resume.md](cases/task-output-cursor-and-resume.md)
- [cases/task-retention-and-eviction.md](cases/task-retention-and-eviction.md)

### Harness.MultiAgent

- [cases/inter-agent-task-notification-routing.md](cases/inter-agent-task-notification-routing.md)
- [cases/teammate-mailbox-delivery.md](cases/teammate-mailbox-delivery.md)
- [cases/teammate-output-not-auto-routed-to-leader.md](cases/teammate-output-not-auto-routed-to-leader.md)
- [cases/viewed-teammate-direct-input.md](cases/viewed-teammate-direct-input.md)

### Harness.ContextEngineering

- [cases/context-planes-and-projection.md](cases/context-planes-and-projection.md)
- [cases/attachment-assembly-order-and-scope.md](cases/attachment-assembly-order-and-scope.md)
- [cases/startup-context-lifecycle.md](cases/startup-context-lifecycle.md)
- [cases/context-provider-vs-assembly-boundary.md](cases/context-provider-vs-assembly-boundary.md)

### Session And Memory

- [cases/chat-session-binding.md](cases/chat-session-binding.md)
- [cases/single-active-harness-lease.md](cases/single-active-harness-lease.md)
- [cases/agent-global-long-memory.md](cases/agent-global-long-memory.md)
- [cases/agents-memory-loading-precedence.md](cases/agents-memory-loading-precedence.md)
- [cases/memory-recall-and-consolidation.md](cases/memory-recall-and-consolidation.md)
- [cases/memory-consolidation-background-safety.md](cases/memory-consolidation-background-safety.md)

### Tools.SkillImport

- [cases/skills-discovery-precedence.md](cases/skills-discovery-precedence.md)
- [cases/skills-activation-wrapping.md](cases/skills-activation-wrapping.md)
- [cases/skills-context-protection.md](cases/skills-context-protection.md)

### Tools.McpClient

- [cases/mcp-tool-adaptation.md](cases/mcp-tool-adaptation.md)
- [cases/mcp-initialize-and-version-negotiation.md](cases/mcp-initialize-and-version-negotiation.md)
- [cases/mcp-streamable-http-post-and-sse.md](cases/mcp-streamable-http-post-and-sse.md)
- [cases/mcp-roots-list-and-list-changed.md](cases/mcp-roots-list-and-list-changed.md)
- [cases/mcp-sampling-with-tools.md](cases/mcp-sampling-with-tools.md)
- [cases/mcp-elicitation-form-vs-url.md](cases/mcp-elicitation-form-vs-url.md)
- [cases/mcp-resource-subscribe-and-list-changed.md](cases/mcp-resource-subscribe-and-list-changed.md)
- [cases/mcp-prompt-pagination-and-get.md](cases/mcp-prompt-pagination-and-get.md)
- [cases/mcp-tool-pagination-and-call.md](cases/mcp-tool-pagination-and-call.md)
- [cases/mcp-auth-discovery-and-scope-upgrade.md](cases/mcp-auth-discovery-and-scope-upgrade.md)
- [cases/mcp-host-extension-skill-discovery.md](cases/mcp-host-extension-skill-discovery.md)

### Tools.PersistedToolResult

- [cases/persisted-tool-result-resume.md](cases/persisted-tool-result-resume.md)

### Harness.PromptCacheStrategy

- [cases/prompt-cache-stable-prefix.md](cases/prompt-cache-stable-prefix.md)
- [cases/prompt-cache-dynamic-suffix.md](cases/prompt-cache-dynamic-suffix.md)
- [cases/prompt-cache-fork-sharing.md](cases/prompt-cache-fork-sharing.md)
- [cases/prompt-cache-break-detection.md](cases/prompt-cache-break-detection.md)
- [cases/prompt-cache-strategy-equivalence.md](cases/prompt-cache-strategy-equivalence.md)

### Sandbox And Orchestration

- [cases/sandbox-deny.md](cases/sandbox-deny.md)
- [cases/cloud-wake-and-reprovision.md](cases/cloud-wake-and-reprovision.md)
- [cases/hosting-profile-equivalence.md](cases/hosting-profile-equivalence.md)

### Golden Artifacts

- [golden/basic-turn.events.json](golden/basic-turn.events.json)
- [golden/tool-call-roundtrip.events.json](golden/tool-call-roundtrip.events.json)
- [golden/requires-action-approval.events.json](golden/requires-action-approval.events.json)
- [golden/session-resume.event-log.json](golden/session-resume.event-log.json)
- [golden/sandbox-deny.events.json](golden/sandbox-deny.events.json)
- [golden/policy-ask-deny-allow.json](golden/policy-ask-deny-allow.json)
- [golden/persisted-tool-result-resume.json](golden/persisted-tool-result-resume.json)
- [golden/agents-memory-loading-precedence.json](golden/agents-memory-loading-precedence.json)
- [golden/memory-recall-and-consolidation.json](golden/memory-recall-and-consolidation.json)
- [golden/memory-consolidation-background-safety.json](golden/memory-consolidation-background-safety.json)
- [golden/skills-discovery-precedence.json](golden/skills-discovery-precedence.json)
- [golden/skills-activation-wrapping.json](golden/skills-activation-wrapping.json)
- [golden/skills-context-protection.json](golden/skills-context-protection.json)
- [golden/mcp-initialize-and-version-negotiation.json](golden/mcp-initialize-and-version-negotiation.json)
- [golden/mcp-sampling-with-tools.json](golden/mcp-sampling-with-tools.json)
- [golden/mcp-auth-discovery-and-scope-upgrade.json](golden/mcp-auth-discovery-and-scope-upgrade.json)
- [golden/cloud-wake-and-reprovision.json](golden/cloud-wake-and-reprovision.json)
- [golden/context-planes-and-projection.json](golden/context-planes-and-projection.json)
- [golden/attachment-assembly-order-and-scope.json](golden/attachment-assembly-order-and-scope.json)
- [golden/prompt-cache-stable-prefix.json](golden/prompt-cache-stable-prefix.json)
- [golden/prompt-cache-dynamic-suffix.json](golden/prompt-cache-dynamic-suffix.json)
- [golden/prompt-cache-fork-sharing.json](golden/prompt-cache-fork-sharing.json)
- [golden/prompt-cache-break-detection.json](golden/prompt-cache-break-detection.json)
- [golden/prompt-cache-strategy-equivalence.json](golden/prompt-cache-strategy-equivalence.json)

## 测试层次

### Canonical Cases

`cases/` 下的文档定义标准场景。

每个场景至少描述：

- preconditions
- ingress
- expected lifecycle 或 expected runtime semantics
- expected persistent effects
- allowed implementation variance
- failure conditions

### Golden Artifacts

`golden/` 下的文件提供可共享的输入输出样本。

这些样本用于：

- replay
- snapshot comparison
- adapter conformance
- cross-host equivalence checks

golden 只保留当前正文仍有明确稳定对象或稳定事件输出的场景。
没有稳定快照价值的场景只保留 canonical case，不强制维护 golden。

### Host Equivalence

同一 canonical case 可以在：

- `Local`
- `Cloud`

两种 host profile 下运行。

允许不同 host 的进程边界、部署形态、存储位置不同，但不允许外部可观察语义漂移。

## 最低覆盖要求

若要宣称实现本规范，至少应通过：

- `basic-turn`
- `tool-call-roundtrip`
- `requires-action-approval`
- `session-resume`

若要宣称支持 `Harness.Permission`，还应通过：

- `policy-ask-deny-allow`
- `permission-rule-precedence-and-shadowing`
- `permission-mode-and-headless-degradation`
- `permission-update-and-scope`

若要宣称支持 `Harness.Task`，还应通过：

- `background-agent`
- `runtime-task-lifecycle`
- `task-output-cursor-and-resume`
- `task-retention-and-eviction`

若要宣称支持 `Harness.MultiAgent`，还应通过：

- `inter-agent-task-notification-routing`
- `teammate-mailbox-delivery`
- `teammate-output-not-auto-routed-to-leader`
- `viewed-teammate-direct-input`

若要宣称支持 `Harness.ContextEngineering` 的完整分层装配语义，还应通过：

- `context-planes-and-projection`
- `attachment-assembly-order-and-scope`
- `startup-context-lifecycle`
- `context-provider-vs-assembly-boundary`

若要宣称支持 `Session.MemoryConsolidation` 或 file-backed memory injection，还应通过：

- `chat-session-binding`
- `single-active-harness-lease`
- `agent-global-long-memory`
- `agents-memory-loading-precedence`
- `memory-recall-and-consolidation`
- `memory-consolidation-background-safety`

若要宣称支持 `Tools.SkillImport`，还应通过：

- `skills-discovery-precedence`
- `skills-activation-wrapping`
- `skills-context-protection`

若要宣称支持 `Tools.McpClient`，还应通过：

- `mcp-tool-adaptation`
- `mcp-initialize-and-version-negotiation`
- `mcp-streamable-http-post-and-sse`
- `mcp-prompt-pagination-and-get`
- `mcp-tool-pagination-and-call`

若要宣称支持 MCP client features，还应通过：

- `mcp-roots-list-and-list-changed`
- `mcp-sampling-with-tools`
- `mcp-elicitation-form-vs-url`

若要宣称支持 MCP resource observation、HTTP auth 或 host extension 映射，还应通过：

- `mcp-resource-subscribe-and-list-changed`
- `mcp-auth-discovery-and-scope-upgrade`
- `mcp-host-extension-skill-discovery`

若要宣称支持 `Tools.PersistedToolResult`，还应通过：

- `persisted-tool-result-resume`

若要宣称支持 `Harness.PromptCacheStrategy`，还应通过：

- `prompt-cache-stable-prefix`
- `prompt-cache-dynamic-suffix`
- `prompt-cache-fork-sharing`
- `prompt-cache-break-detection`
- `prompt-cache-strategy-equivalence`

若要宣称支持 `Sandbox` denial boundary，还应通过：

- `sandbox-deny`

若要宣称支持 `Orchestration.ManagedControlPlane` 或 Cloud hosting profile，还应通过：

- `cloud-wake-and-reprovision`
- `hosting-profile-equivalence`

## 维护规则

- case 只要失去正文规范锚点，就应删除，而不是继续靠 `conformance` 自身自证存在
- 正文语义变化若影响验收口径，必须同步更新 case 与对应 golden
- 若某次正文改动没有影响 conformance，交付说明中必须显式注明“无 conformance 影响”及理由

## 规范结论

- `conformance` 测试的重点是行为语义，而不是内部实现结构
- `golden artifacts` 应保持语言无关、宿主无关
- host extension 可以有独立用例，但不能被误判为 MCP core 最低门槛
