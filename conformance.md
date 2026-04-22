# Conformance

## 目标

本文件定义各语言 agent 实现的共享合规要求。

不同实现可以使用不同的并发模型、存储介质、传输协议适配和工程组织方式，但不能改变当前 `agent-spec/` 已定义的稳定行为语义。

合规声明必须按两层给出：

- `核心合规`
  只覆盖五个顶层模块下的最小稳定行为
- `扩展合规`
  只在实现对外宣称支持某个标准扩展时才适用

## 一致性范围

需要跨语言保持一致的内容：

- runtime event 与 terminal state 的外部可观察语义
- tool validation / policy / execution / persistence 的分层语义
- permission mode / rule / approval / headless degradation 语义
- context governance 与 prompt cache strategy 的触发与恢复语义
- context planes、attachment ordering、startup context lifecycle、provider/assembly boundary、context editing 分层退化与恢复语义
- session transcript / checkpoint / resume / short-term continuity 语义
- session working state、observer-facing lifecycle state 与 sidechain transcript 的分层语义
- durable memory overview / recall / consolidation / branch tracing 语义
- auto-memory 的 resident index、bounded recall 与 write-path layering 语义
- auto-memory 的 entrypoint index、topic payload、header manifest 与 payload-selection layering 语义
- durable memory taxonomy 中 `user / feedback / project / reference` 的类型边界、scope 与 staleness 语义
- durable memory taxonomy 与 `user / project / team / agent / local` overlays 的正交语义
- task lifecycle / output cursor / notification / retention 语义
- local multi-agent delegation / mailbox / viewed transcript / notification routing 语义
- MCP lifecycle / transport / capability negotiation / runtime adaptation 语义
- `Local-first` 默认实现下的 `Cloud-compatible` 部署边界语义

允许实现差异的内容：

- 类名、函数名、文件组织
- 线程、协程、异步任务实现方式
- 本地缓存、序列化、日志框架
- 具体 provider、HTTP client、存储后端

## 核心合规

### Runtime

实现必须对外暴露语义等价的 runtime 事件分类与终态分类。

至少应能稳定区分：

- assistant 输出增量
- tool 启动、进展、结果
- requires-action 阻塞
- turn 正常完成
- turn 失败、用户中止、权限阻塞、预算终止

事件字段名可以随语言习惯调整，但事件含义不能变化。

实现还必须保持：

- runtime core 消费 normalized ingress，而不是 raw channel payload
- runtime 拥有 session binding / switching orchestration，但不拥有 session durable store
- 一次 turn 可以跨多次 provider call 与 continuation 完成
- `tool loop` 与 `recursive follow-up` 语义可区分
- `tool_use / tool_result` pairing 与 ordering 语义稳定

若实现宣称支持完整的 runtime state projection，还必须保持：

- session / turn / task 的 observer-facing facts 可稳定对外暴露
- task-derived projection 不改变 `Harness.Task` 对 task lifecycle 的所有权
- projection channel 与 transcript / message stream 语义分离

### Harness 与 Permission

实现必须保持：

- context governance 可被 runtime 触发
- tool visibility 与 execute-time permission 分层
- `allow / deny / ask` 为稳定决策面
- `ask -> requires_action` 为稳定阻塞投影
- headless / background / worker 场景不会产生不可达 approval
- static rule、tool-specific check、safety check、mode transform、approval route 的先后语义稳定
- `Local` 与 `Cloud` 下的 harness 都保持为一个完整整体
- `Local` 下 direct-call、`Cloud` 下 RPC 只改变 binding，不改变 harness 内部语义边界

### Session

实现必须保持：

- append-only transcript 或语义等价 durable event log
- `SessionCheckpoint` / `ResumeSnapshot` 的 durable progress 含义
- `WorkingState` 与 `SessionLifecycleState` 的边界可区分
- `RequiresAction` 可恢复
- harness 或 sandbox 替换后仍可基于 session 继续
- `1 Chat = 1 Session`
- `1 Session = 1 Short-Term Memory`
- sidechain / subagent transcript 与主 transcript 的追溯关系稳定

### Tools

实现必须保持：

- `ToolDefinition` 的校验失败、权限失败、执行失败三类语义可区分
- batch executor 与 streaming executor 的 terminal state / error class 语义一致
- context mutation 的 staged / committed / discarded 语义一致
- `skill`、`tool`、`command`、`mcp` 的角色边界稳定
- MCP compatibility 不能被简化成“支持一批远端 tools”
- plugin package / source loading / runtime delegation 与 execution capability 边界稳定

### Sandbox

实现必须保持：

- sandbox capability / security boundary 为独立执行边界
- sandbox violation 不会被伪装成普通 tool failure
- permission runtime 可消费 sandbox 输入，但不替代 sandbox 本体

### Orchestration

若实现提供 Cloud hosting profile 或托管式 agent：

- wake / resume 作用于既有 session binding
- reprovision 作用于 execution environment，不改变 session identity
- many brains / many hands / lazy provisioning 不改变模块边界
- durable session history 与 execution environment lifecycle 解耦

## 标准扩展合规

### Harness.PromptCacheStrategy

若实现宣称支持 prompt-cache-aware harness，还必须保持：

- stable prefix 语义一致
- dynamic suffix 语义一致
- fork sharing 语义一致
- cache break detection 语义一致
- strategy adapter 不改变对外 context governance 语义

### Harness.ContextEngineering

若实现宣称支持完整的 `Harness.ContextEngineering` 分层装配语义，还必须保持：

- context planes 分层稳定
- attachment ordering 与 scope 语义 deterministic
- startup-only context 与 per-turn context 生命周期分离
- provider 只产出 fragment，assembly 与 governance 保持独立职责
- context editing 支持从低损 externalization / projection 到高损 rewrite 的分层退化
- externalized payload 在 compact / resume 后保持稳定 evidence ref
- working-view projection 与 transcript-visible compaction rewrite 语义分离
- `AGENTS.md` / `rules` / include expansion 一类 instruction markdown loading 属于 harness-level context input，而不是 durable memory recall

### Harness.Task

若实现宣称支持 task 子系统，还必须保持：

- task 至少可区分 `pending / running / completed / failed / killed`
- `output_ref + cursor` 为稳定接口
- progress 与 terminal notification 分离
- terminal notification 可去重
- restore 后可恢复 task status、output handle、terminal-like stop reason
- retention / eviction 不得破坏既有 terminal 事实与输出追溯

### Harness.MultiAgent

若实现宣称支持 local multi-agent coordination，还必须保持：

- delegated subagent、background agent、teammate 的角色边界稳定
- task 回流、mailbox、viewed transcript、direct input 的通道语义可区分
- notification、mailbox、viewed input 的 recipient scope deterministic
- worker identity 稳定，不因前后台切换或观察方式改变

### Harness.AgentProfiles

若实现宣称支持 assistant-mode 等长期运行的 `Harness.AgentProfiles`，还必须保持：

- profile 语义是对 gateway、task、multi-agent、session.memory 等子域能力的组合，而不是 ownership 替代
- assistant agent profile 的主交互面偏响应式
- assistant agent profile 可以采用 append-first / consolidate-later 的 memory operating mode
- continuity / viewer attach 与上述 memory operating mode 兼容

### Harness.Permission

若实现宣称支持完整 permission rule system，还必须保持：

- `PermissionRule` 为结构化对象
- deny 优先于 allow
- rule precedence / shadowing deterministic
- permission update 的作用域与生效边界稳定
- `dont_ask` / `auto` / `plan` 等模式不会绕过显式 deny 或 bypass-immune safety check

### Session.MemoryConsolidation

若实现宣称支持 session memory 或 consolidation，还必须保持：

- short-term session continuity 与 transcript 分离
- durable memory recall 不承担 restore 责任
- durable memory 必须区分 architecture、recall、write/consolidation 与 scope overlay
- durable memory 具有常驻索引与 bounded recall 的语义分层
- auto-memory 作为默认 durable runtime 时，resident index、background extraction、dream consolidation 与 assistant-style append-first mode 的分层语义稳定
- auto-memory 作为默认 durable runtime 时，entrypoint index、topic payload、header manifest 与 payload-selection layering 语义稳定
- durable memory taxonomy 中 `user / feedback / project / reference` 的类型边界稳定
- durable memory 的 taxonomy axis 与 scope overlay axis 必须可区分
- `feedback` 同时覆盖 correction 与 validated success，并保持 private-vs-shared guidance 语义
- `project` 允许绝对时间锚点与 staleness 建模
- `reference` 作为 pointer memory 与 `user` 作为 private personalization layer 的边界稳定
- `user / project / team / agent / local` overlays 只改变 binding、sharing 与 sync surface，不定义新的 durable plane
- consolidation 不得破坏 resume 语义
- branch / sidechain transcript 与父 session 的追溯关系一致
- 同一 agent 下多个 session 可共享同一个 global long-term memory
- 同一 session 任一时刻只能有一个 active harness lease

### Tools.SkillImport

若实现宣称支持 Agent Skills specification 导入，还必须保持：

- `SKILL.md` 目录导入语义一致
- discovery precedence / shadowing deterministic
- catalog disclosure 与 activation disclosure 分层一致
- activation 结果具备可 dedupe、可 compaction-protect 的稳定语义

### Tools.PluginPackaging

若实现宣称支持 plugin package / source loading 语义，还必须保持：

- plugin 是 package / source loading / runtime delegation layer
- plugin 不得被等同于 tool / skill / command / mcp
- runtime startup / refresh 时的 plugin discovery、enablement、source precedence 语义稳定
- commands / skills / hooks / mcp servers / agents / settings 等 component set 必须委托给既有子系统，而不是由 plugin 自行执行

### Tools.McpClient

若实现宣称支持 MCP `2025-11-25` compatibility，还必须保持：

- `initialize -> initialized` 生命周期闭合
- version / capability negotiation 结果稳定
- `stdio` 与 `Streamable HTTP` 的能力语义等价
- `roots / sampling / elicitation` 仅在协商成功时生效
- `tools / prompts / resources` 的 pagination 与 list-changed 语义一致
- `resources/subscribe` 与 `notifications/resources/updated` 的观察语义一致

以下内容只能算 host extension，不算 MCP core requirement：

- `mcp skill`
- bundle host / install / trust flow
- UI-specific affordance

### Tools.PersistedToolResult

若实现宣称支持大结果外存化，还必须保持：

- `PersistedToolResultRef` 可序列化
- preview 与 full result reference 的关系一致
- resume / compact / branch restore 后引用仍稳定到同一 `tool_use_id`

### Orchestration.ManagedControlPlane

若实现宣称支持 managed-agent control plane，还必须保持：

- harness crash recovery 可基于 session log 恢复
- execution target 失效后可 reprovision 并继续或结构化失败回写
- many brains / many hands 不依赖单一宠物进程
- lazy provisioning 不为无 execution 需求的 turn 引入额外冷启动语义漂移

## Canonical Cases

共享合规场景索引见 [conformance/README.md](conformance/README.md)。

最低核心覆盖要求：

- [conformance/cases/basic-turn.md](conformance/cases/basic-turn.md)
- [conformance/cases/tool-call-roundtrip.md](conformance/cases/tool-call-roundtrip.md)
- [conformance/cases/requires-action-approval.md](conformance/cases/requires-action-approval.md)
- [conformance/cases/session-resume.md](conformance/cases/session-resume.md)
- [conformance/cases/runtime-session-binding-and-switching.md](conformance/cases/runtime-session-binding-and-switching.md)
- [conformance/cases/runtime-turn-state-machine.md](conformance/cases/runtime-turn-state-machine.md)
- [conformance/cases/turn-loop-iteration-and-handoff.md](conformance/cases/turn-loop-iteration-and-handoff.md)
- [conformance/cases/runtime-tool-use-and-tool-result-pairing.md](conformance/cases/runtime-tool-use-and-tool-result-pairing.md)
- [conformance/cases/gateway-ingress-vs-runtime-core-boundary.md](conformance/cases/gateway-ingress-vs-runtime-core-boundary.md)
- [conformance/cases/harness-deployment-binding-equivalence.md](conformance/cases/harness-deployment-binding-equivalence.md)

按标准扩展追加覆盖：

- `Harness.Permission`
  - [conformance/cases/policy-ask-deny-allow.md](conformance/cases/policy-ask-deny-allow.md)
  - [conformance/cases/permission-rule-precedence-and-shadowing.md](conformance/cases/permission-rule-precedence-and-shadowing.md)
  - [conformance/cases/permission-mode-and-headless-degradation.md](conformance/cases/permission-mode-and-headless-degradation.md)
  - [conformance/cases/permission-update-and-scope.md](conformance/cases/permission-update-and-scope.md)
- `Harness.Task`
  - [conformance/cases/background-agent.md](conformance/cases/background-agent.md)
  - [conformance/cases/runtime-task-lifecycle.md](conformance/cases/runtime-task-lifecycle.md)
  - [conformance/cases/task-output-cursor-and-resume.md](conformance/cases/task-output-cursor-and-resume.md)
  - [conformance/cases/task-retention-and-eviction.md](conformance/cases/task-retention-and-eviction.md)
- `Harness.MultiAgent`
  - [conformance/cases/inter-agent-task-notification-routing.md](conformance/cases/inter-agent-task-notification-routing.md)
  - [conformance/cases/teammate-mailbox-delivery.md](conformance/cases/teammate-mailbox-delivery.md)
  - [conformance/cases/teammate-output-not-auto-routed-to-leader.md](conformance/cases/teammate-output-not-auto-routed-to-leader.md)
  - [conformance/cases/viewed-teammate-direct-input.md](conformance/cases/viewed-teammate-direct-input.md)
- `Harness.AgentProfiles`
  - [conformance/cases/assistant-agent-profile-background-responsiveness.md](conformance/cases/assistant-agent-profile-background-responsiveness.md)
  - [conformance/cases/assistant-agent-profile-memory-and-continuity.md](conformance/cases/assistant-agent-profile-memory-and-continuity.md)
- `Harness.ContextEngineering`
  - [conformance/cases/context-planes-and-projection.md](conformance/cases/context-planes-and-projection.md)
  - [conformance/cases/attachment-assembly-order-and-scope.md](conformance/cases/attachment-assembly-order-and-scope.md)
  - [conformance/cases/startup-context-lifecycle.md](conformance/cases/startup-context-lifecycle.md)
  - [conformance/cases/context-provider-vs-assembly-boundary.md](conformance/cases/context-provider-vs-assembly-boundary.md)
  - [conformance/cases/instruction-markdown-loading-precedence.md](conformance/cases/instruction-markdown-loading-precedence.md)
  - [conformance/cases/instruction-rules-and-include-expansion.md](conformance/cases/instruction-rules-and-include-expansion.md)
- `Session.MemoryConsolidation` / `DurableMemory`
  - [conformance/cases/agent-global-long-memory.md](conformance/cases/agent-global-long-memory.md)
  - [conformance/cases/durable-memory-index-and-bounded-recall.md](conformance/cases/durable-memory-index-and-bounded-recall.md)
  - [conformance/cases/auto-memory-resident-index-and-bounded-recall.md](conformance/cases/auto-memory-resident-index-and-bounded-recall.md)
  - [conformance/cases/durable-memory-type-boundaries-and-exclusions.md](conformance/cases/durable-memory-type-boundaries-and-exclusions.md)
  - [conformance/cases/feedback-memory-success-correction-and-scope.md](conformance/cases/feedback-memory-success-correction-and-scope.md)
  - [conformance/cases/project-memory-staleness-and-absolute-date.md](conformance/cases/project-memory-staleness-and-absolute-date.md)
  - [conformance/cases/reference-vs-user-memory-pointer-and-private-boundary.md](conformance/cases/reference-vs-user-memory-pointer-and-private-boundary.md)
  - [conformance/cases/memory-recall-and-consolidation.md](conformance/cases/memory-recall-and-consolidation.md)
  - [conformance/cases/memory-consolidation-background-safety.md](conformance/cases/memory-consolidation-background-safety.md)
  - [conformance/cases/auto-memory-write-paths-and-consolidation.md](conformance/cases/auto-memory-write-paths-and-consolidation.md)
  - [conformance/cases/chat-session-binding.md](conformance/cases/chat-session-binding.md)
  - [conformance/cases/single-active-harness-lease.md](conformance/cases/single-active-harness-lease.md)
- `Tools.SkillImport`
  - [conformance/cases/skills-discovery-precedence.md](conformance/cases/skills-discovery-precedence.md)
  - [conformance/cases/skills-activation-wrapping.md](conformance/cases/skills-activation-wrapping.md)
  - [conformance/cases/skills-context-protection.md](conformance/cases/skills-context-protection.md)
- `Tools.McpClient`
  - [conformance/cases/mcp-tool-adaptation.md](conformance/cases/mcp-tool-adaptation.md)
  - [conformance/cases/mcp-initialize-and-version-negotiation.md](conformance/cases/mcp-initialize-and-version-negotiation.md)
  - [conformance/cases/mcp-streamable-http-post-and-sse.md](conformance/cases/mcp-streamable-http-post-and-sse.md)
  - [conformance/cases/mcp-prompt-pagination-and-get.md](conformance/cases/mcp-prompt-pagination-and-get.md)
  - [conformance/cases/mcp-tool-pagination-and-call.md](conformance/cases/mcp-tool-pagination-and-call.md)
  - [conformance/cases/mcp-roots-list-and-list-changed.md](conformance/cases/mcp-roots-list-and-list-changed.md)
  - [conformance/cases/mcp-sampling-with-tools.md](conformance/cases/mcp-sampling-with-tools.md)
  - [conformance/cases/mcp-elicitation-form-vs-url.md](conformance/cases/mcp-elicitation-form-vs-url.md)
  - [conformance/cases/mcp-resource-subscribe-and-list-changed.md](conformance/cases/mcp-resource-subscribe-and-list-changed.md)
  - [conformance/cases/mcp-auth-discovery-and-scope-upgrade.md](conformance/cases/mcp-auth-discovery-and-scope-upgrade.md)
  - [conformance/cases/mcp-host-extension-skill-discovery.md](conformance/cases/mcp-host-extension-skill-discovery.md)
- `Tools.PersistedToolResult`
  - [conformance/cases/persisted-tool-result-resume.md](conformance/cases/persisted-tool-result-resume.md)
- `Harness.PromptCacheStrategy`
  - [conformance/cases/prompt-cache-stable-prefix.md](conformance/cases/prompt-cache-stable-prefix.md)
  - [conformance/cases/prompt-cache-dynamic-suffix.md](conformance/cases/prompt-cache-dynamic-suffix.md)
  - [conformance/cases/prompt-cache-fork-sharing.md](conformance/cases/prompt-cache-fork-sharing.md)
  - [conformance/cases/prompt-cache-break-detection.md](conformance/cases/prompt-cache-break-detection.md)
  - [conformance/cases/prompt-cache-strategy-equivalence.md](conformance/cases/prompt-cache-strategy-equivalence.md)
- `Sandbox`
  - [conformance/cases/sandbox-deny.md](conformance/cases/sandbox-deny.md)
- `Orchestration.ManagedControlPlane`
  - [conformance/cases/cloud-wake-and-reprovision.md](conformance/cases/cloud-wake-and-reprovision.md)
  - [conformance/cases/hosting-profile-equivalence.md](conformance/cases/hosting-profile-equivalence.md)

## 维护要求

修改以下任一内容时，必须同步评估并维护 conformance 资产：

- 模块职责边界
- 最小稳定接口
- canonical object model 字段或语义
- task、permission、session、memory、MCP、prompt cache、hosting profile 的行为语义
- 最低覆盖要求或扩展能力宣称条件

维护时必须同步检查：

- [conformance/README.md](conformance/README.md)
- [conformance/cases/](conformance/cases/)
- [conformance/golden/](conformance/golden/)
- [test-artifacts.md](test-artifacts.md)

若正文调整没有影响 canonical case 或 golden，交付说明中必须显式写明“无 conformance 影响”及理由。

## 规范结论

- 合规测试验证的是外部行为语义，不是内部代码结构
- 核心合规与扩展合规必须分开声明
- 只有当前规范正文明确承载的能力，才应保留对应 canonical case
- host extension 可以有独立 case，但不得被误报为 MCP core 或第六个顶层模块
