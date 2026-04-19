# Conformance Cases

## 目标

本目录提供跨语言、跨宿主共享的合规测试资产。

这些资产不绑定具体语言实现，也不绑定单一宿主形态，而是用于验证：

- `Local / Cloud` 两种 host 下的外部行为是否等价
- 不同语言 agent 实现是否保持相同的 runtime 语义
- `Harness / Session / Tools / Sandbox / Orchestration` 五个模块的接口语义是否被正确实现

## 目录结构

- [cases/basic-turn.md](cases/basic-turn.md)
- [cases/tool-call-roundtrip.md](cases/tool-call-roundtrip.md)
- [cases/policy-ask-deny-allow.md](cases/policy-ask-deny-allow.md)
- [cases/permission-rule-precedence-and-shadowing.md](cases/permission-rule-precedence-and-shadowing.md)
- [cases/permission-mode-and-headless-degradation.md](cases/permission-mode-and-headless-degradation.md)
- [cases/permission-update-and-scope.md](cases/permission-update-and-scope.md)
- [cases/requires-action-approval.md](cases/requires-action-approval.md)
- [cases/session-resume.md](cases/session-resume.md)
- [cases/chat-session-binding.md](cases/chat-session-binding.md)
- [cases/single-active-harness-lease.md](cases/single-active-harness-lease.md)
- [cases/agent-global-long-memory.md](cases/agent-global-long-memory.md)
- [cases/persisted-tool-result-resume.md](cases/persisted-tool-result-resume.md)
- [cases/skills-discovery-precedence.md](cases/skills-discovery-precedence.md)
- [cases/skills-activation-wrapping.md](cases/skills-activation-wrapping.md)
- [cases/skills-context-protection.md](cases/skills-context-protection.md)
- [cases/background-agent.md](cases/background-agent.md)
- [cases/runtime-task-lifecycle.md](cases/runtime-task-lifecycle.md)
- [cases/task-output-cursor-and-resume.md](cases/task-output-cursor-and-resume.md)
- [cases/task-retention-and-eviction.md](cases/task-retention-and-eviction.md)
- [cases/inter-agent-task-notification-routing.md](cases/inter-agent-task-notification-routing.md)
- [cases/teammate-mailbox-delivery.md](cases/teammate-mailbox-delivery.md)
- [cases/teammate-output-not-auto-routed-to-leader.md](cases/teammate-output-not-auto-routed-to-leader.md)
- [cases/viewed-teammate-direct-input.md](cases/viewed-teammate-direct-input.md)
- [cases/hosting-profile-equivalence.md](cases/hosting-profile-equivalence.md)
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
- [cases/sandbox-deny.md](cases/sandbox-deny.md)
- [cases/memory-recall-and-consolidation.md](cases/memory-recall-and-consolidation.md)
- [cases/agents-memory-loading-precedence.md](cases/agents-memory-loading-precedence.md)
- [cases/memory-consolidation-background-safety.md](cases/memory-consolidation-background-safety.md)
- [cases/cloud-wake-and-reprovision.md](cases/cloud-wake-and-reprovision.md)
- [cases/prompt-cache-stable-prefix.md](cases/prompt-cache-stable-prefix.md)
- [cases/prompt-cache-dynamic-suffix.md](cases/prompt-cache-dynamic-suffix.md)
- [cases/prompt-cache-fork-sharing.md](cases/prompt-cache-fork-sharing.md)
- [cases/prompt-cache-break-detection.md](cases/prompt-cache-break-detection.md)
- [cases/prompt-cache-strategy-equivalence.md](cases/prompt-cache-strategy-equivalence.md)
- [golden/basic-turn.events.json](golden/basic-turn.events.json)
- [golden/tool-call-roundtrip.events.json](golden/tool-call-roundtrip.events.json)
- [golden/requires-action-approval.events.json](golden/requires-action-approval.events.json)
- [golden/session-resume.event-log.json](golden/session-resume.event-log.json)
- [golden/sandbox-deny.events.json](golden/sandbox-deny.events.json)
- [golden/memory-recall-and-consolidation.json](golden/memory-recall-and-consolidation.json)
- [golden/agents-memory-loading-precedence.json](golden/agents-memory-loading-precedence.json)
- [golden/policy-ask-deny-allow.json](golden/policy-ask-deny-allow.json)
- [golden/persisted-tool-result-resume.json](golden/persisted-tool-result-resume.json)
- [golden/memory-consolidation-background-safety.json](golden/memory-consolidation-background-safety.json)
- [golden/skills-discovery-precedence.json](golden/skills-discovery-precedence.json)
- [golden/skills-activation-wrapping.json](golden/skills-activation-wrapping.json)
- [golden/skills-context-protection.json](golden/skills-context-protection.json)
- [golden/cloud-wake-and-reprovision.json](golden/cloud-wake-and-reprovision.json)
- [golden/prompt-cache-stable-prefix.json](golden/prompt-cache-stable-prefix.json)
- [golden/prompt-cache-dynamic-suffix.json](golden/prompt-cache-dynamic-suffix.json)
- [golden/prompt-cache-fork-sharing.json](golden/prompt-cache-fork-sharing.json)
- [golden/prompt-cache-break-detection.json](golden/prompt-cache-break-detection.json)
- [golden/prompt-cache-strategy-equivalence.json](golden/prompt-cache-strategy-equivalence.json)
- [golden/mcp-initialize-and-version-negotiation.json](golden/mcp-initialize-and-version-negotiation.json)
- [golden/mcp-sampling-with-tools.json](golden/mcp-sampling-with-tools.json)
- [golden/mcp-auth-discovery-and-scope-upgrade.json](golden/mcp-auth-discovery-and-scope-upgrade.json)

## 测试层次

### 1. Canonical Cases

`cases/` 下的文档定义标准场景。

每个场景至少描述：

- preconditions
- ingress
- expected lifecycle
- expected runtime events
- expected transcript or event log effects
- allowed implementation variance

### 2. Golden Artifacts

`golden/` 下的文件提供可共享的输入输出样本。

这些样本用于：

- replay
- snapshot comparison
- adapter conformance
- cross-host equivalence checks

### 3. Host Equivalence

同一 canonical case 可以在：

- `Local`
- `Cloud`

两种 host profile 下运行。

允许不同 host 的进程边界、部署形态、存储位置不同，但不允许外部可观察语义漂移。

## 最低覆盖要求

一个语言 agent 实现若要宣称实现本规范，至少应能通过：

- `basic-turn`
- `tool-call-roundtrip`
- `requires-action-approval`
- `session-resume`

若要宣称支持 agent orchestration，还应通过：

- `background-agent`

若要宣称支持 `Harness.Task` 或 task lifecycle 规范，还应通过：

- `runtime-task-lifecycle`
- `task-output-cursor-and-resume`
- `task-retention-and-eviction`

若要宣称支持 `Harness.MultiAgent` 或 local multi-agent 规范，还应通过：

- `inter-agent-task-notification-routing`
- `teammate-mailbox-delivery`
- `teammate-output-not-auto-routed-to-leader`
- `viewed-teammate-direct-input`

若要宣称支持跨宿主一致性，还应通过：

- `hosting-profile-equivalence`

若要宣称支持产品级 `agent / gateway / session / memory` 绑定关系，还应通过：

- `chat-session-binding`
- `single-active-harness-lease`
- `agent-global-long-memory`

若要宣称支持 MCP compatibility，还应通过：

- `mcp-tool-adaptation`
- `mcp-initialize-and-version-negotiation`
- `mcp-streamable-http-post-and-sse`
- `mcp-prompt-pagination-and-get`
- `mcp-tool-pagination-and-call`

若要宣称支持 MCP client features，还应通过：

- `mcp-roots-list-and-list-changed`
- `mcp-sampling-with-tools`
- `mcp-elicitation-form-vs-url`

若要宣称支持 MCP resource observation 或 HTTP auth，还应通过：

- `mcp-resource-subscribe-and-list-changed`
- `mcp-auth-discovery-and-scope-upgrade`

若要宣称支持 host-specific MCP extensions，还应通过：

- `mcp-host-extension-skill-discovery`

若要宣称支持 Agent Skills specification 导入，还应通过：

- `skills-discovery-precedence`
- `skills-activation-wrapping`
- `skills-context-protection`

若要宣称支持 sandbox、memory consolidation、cloud orchestration profile，还应通过：

- `sandbox-deny`
- `memory-recall-and-consolidation`
- `agents-memory-loading-precedence`
- `memory-consolidation-background-safety`
- `cloud-wake-and-reprovision`

若要宣称支持结构化 policy engine，还应通过：

- `policy-ask-deny-allow`

若要宣称支持 `Harness.Permission` 或完整 permission rule system，还应通过：

- `policy-ask-deny-allow`
- `permission-rule-precedence-and-shadowing`
- `permission-mode-and-headless-degradation`
- `permission-update-and-scope`

若要宣称支持大结果外存化与稳定恢复，还应通过：

- `persisted-tool-result-resume`

若要宣称支持 prompt-cache-aware harness，还应通过：

- `prompt-cache-stable-prefix`
- `prompt-cache-dynamic-suffix`
- `prompt-cache-fork-sharing`
- `prompt-cache-break-detection`

若要宣称支持跨 provider prompt cache strategy adapter，还应通过：

- `prompt-cache-strategy-equivalence`

## 规范结论

- `conformance` 测试的重点是行为语义，而不是内部实现结构
- `golden artifacts` 应保持语言无关、宿主无关
- host profile 可以改变部署方式，但不能改变规范定义的外部行为
