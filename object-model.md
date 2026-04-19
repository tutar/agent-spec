# Canonical Object Model

## 目标

本文件定义跨语言共享的核心标准对象。

这些对象不是某个语言 agent 实现的具体类型，而是所有实现都应保持语义一致的 canonical model。

## 设计原则

- 对象语义稳定优先于字段命名一致
- 允许语言实现调整大小写、类型系统映射和序列化库
- 不允许改变对象职责和状态含义
- 对象应最小但足以承载跨模块协作

## 规范位置

本文件定义的是跨模块共享对象，不是某个模块 README 中局部接口的替代品。

推荐做法是：

- 在模块 README 中定义“谁拥有该对象、何时产生、谁消费它”
- 在本文件中定义对象的最小共享字段与一致性约束

## RuntimeEvent

用于描述 runtime 对外发射的标准事件。

推荐最小字段：

```text
RuntimeEvent
  - event_type
  - event_id
  - timestamp
  - session_id
  - agent_id?
  - task_id?
  - turn_id?
  - causation_id?
  - correlation_id?
  - payload
```

推荐 `event_type` 至少包括：

- `request_started`
- `assistant_delta`
- `assistant_message`
- `tool_started`
- `tool_progress`
- `tool_result`
- `tool_failed`
- `tool_cancelled`
- `requires_action`
- `task_notification`
- `turn_completed`
- `turn_failed`
- `turn_cancelled`
- `turn_aborted`
- `turn_timed_out`

约束：

- `RuntimeEvent` 是外部稳定协议，不等于内部消息对象
- `event_type` 应与 [harness/runtime-core/failure-and-terminal-states.md](harness/runtime-core/failure-and-terminal-states.md) 中的终止态和错误语义兼容
- `causation_id / correlation_id` 用于把 event 与 tool call、task、request 或外部动作关联起来

推荐补充事件：

- `session_checkpoint_committed`
- `session_state_changed`
- `memory_updated`
- `compact_boundary_created`

## TerminalState

用于表达一次 turn 或 task 的终止结果。

推荐最小字段：

```text
TerminalState
  - status
  - reason?
  - retryable
  - error?
  - completed_at
  - summary?
```

推荐 `status` 至少包括：

- `completed`
- `failed`
- `cancelled`
- `requires_action`
- `aborted`
- `timed_out`

约束：

- `status` 必须与 [harness/runtime-core/failure-and-terminal-states.md](harness/runtime-core/failure-and-terminal-states.md) 中的统一终止态保持一致
- `retryable` 必须显式
- `error` 如存在，应使用统一 `AgentError`

一个实现即使内部没有 `TerminalState` 结构体，也必须能对外给出语义等价对象。

## AgentError

用于表达跨 runtime / tool / task / session 的统一错误对象。

```text
AgentError
  - class
  - message
  - retryable
  - terminal
  - source: model | tool | task | session | sandbox | transport | policy
  - code?
  - details?
```

推荐 `class` 至少包括：

- `auth_error`
- `permission_denied`
- `rate_limited`
- `context_too_large`
- `output_limit`
- `validation_error`
- `tool_protocol_error`
- `dependency_unavailable`
- `network_transient`
- `execution_target_failed`
- `conflict`
- `not_found`
- `internal_error`
- `non_retryable`

## RequiresAction

用于表达 runtime 被结构化阻塞的状态。

推荐最小字段：

```text
RequiresAction
  - action_type
  - session_id
  - agent_id?
  - task_id?
  - terminal_state: requires_action
  - tool_name?
  - description
  - input?
  - request_id
  - action_ref?
  - policy_decision_id?
  - resumable: boolean
  - details?
```

推荐 `action_type` 至少包括：

- `approval`
- `user_input`
- `selection`
- `plan_review`
- `external_resume`

约束：

- `RequiresAction` 是结构化阻塞对象，不是错误对象
- 其语义必须与 [session/lifecycle-state.md](session/lifecycle-state.md) 以及 [tools/tool-model/policy-engine.md](tools/tool-model/policy-engine.md) 一致

## PolicyDecision

用于表达工具执行前的结构化策略判定。

推荐最小字段：

```text
PolicyDecision
  - decision: allow | deny | ask | passthrough
  - reason
  - explanation?
  - requires_action_ref?
  - policy_source?
  - audit_metadata?
```

约束：

- `passthrough` 仅允许策略链内部使用
- 对外最终结果只能是 `allow / deny / ask`
- `ask` 最终应能映射到 canonical `RequiresAction`
- `deny` 最终应能映射到 canonical `AgentError(permission_denied)`

## PermissionMode

用于表达 permission runtime 的当前授权模式。

推荐最小字段：

```text
PermissionMode
  - value: default | accept_edits | plan | bypass_permissions | dont_ask | auto
```

约束：

- `PermissionMode` 是结构化语义，不应被散落在自由文本配置中
- `dont_ask` 表示 ask-to-deny 的运行时模式，而不是一个 UI 开关
- `auto` 表示 automated authorization mode，而不是“默认允许所有动作”

## PermissionRule

用于表达结构化 permission rule。

推荐最小字段：

```text
PermissionRule
  - source
  - behavior: allow | deny | ask
  - rule_value
```

```text
PermissionRuleValue
  - tool_name
  - rule_content?
```

约束：

- `rule_content` 为空时表示 tool-wide rule
- source、behavior 与 value 必须同时可审计
- deny / ask / allow 的优先级应由 permission runtime 稳定定义

## PermissionContext

用于表达当前 turn / worker / task 所处的 permission environment。

推荐最小字段：

```text
PermissionContext
  - mode
  - additional_working_directories
  - always_allow_rules
  - always_deny_rules
  - always_ask_rules
  - should_avoid_permission_prompts
  - await_automated_checks_before_dialog
  - pre_plan_mode?
```

约束：

- `PermissionContext` 是运行时对象，不等于持久化 settings 文件
- working-directory 与 rule set 必须进入同一 authorization 平面
- headless / background / worker 场景必须通过 context 显式表达

## PermissionUpdate

用于表达 permission configuration 的结构化变更。

推荐最小字段：

```text
PermissionUpdate
  - type
  - destination
  - payload
```

推荐 `type` 至少包括：

- `add_rules`
- `replace_rules`
- `remove_rules`
- `set_mode`
- `add_directories`
- `remove_directories`

约束：

- session update 与 persisted update 应能被稳定区分
- 只读 source 不应被误当成可写 destination
- `PermissionUpdate` 必须可审计、可序列化

## PermissionDecisionReason

用于表达 permission decision 的解释对象。

推荐最小字段：

```text
PermissionDecisionReason
  - type
  - details
```

推荐 `type` 至少包括：

- `rule`
- `mode`
- `hook`
- `classifier`
- `working_dir`
- `safety_check`
- `async_agent`
- `sandbox_override`
- `other`

约束：

- reason 必须可解释、可审计
- tool-specific denial、mode transform 与 automated checks 应共享同一解释平面

## SessionCheckpoint

用于表达 session durable progress 的标准衔接对象。

推荐最小字段：

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
  - event_count?
  - branch_id?
```

约束：

- `SessionCheckpoint` 代表 durable progress，而不是当前内存态
- `cursor` 必须足以支持 `resume`、replay 或增量读取
- 对于 sidechain / branch transcript，`branch_id` 或语义等价字段必须可追溯父 session

## ResumeSnapshot

用于表达 `wake` / `resume` 时恢复 working state 的结构化快照。

推荐最小字段：

```text
ResumeSnapshot
  - session_id
  - wake_id
  - checkpoint
  - runtime_state
  - working_state
  - pending_actions?
  - memory_refs?
  - branch_refs?
```

约束：

- `ResumeSnapshot` 不等于完整 transcript
- 它必须足以让新的 harness 或 execution substrate 在不依赖旧进程内状态的情况下继续执行
- `pending_actions` 如存在，应与 `RequiresAction` 语义兼容

## DurableMemoryRecord

用于表达 durable memory 的最小共享对象。

推荐最小字段：

```text
DurableMemoryRecord
  - memory_id?
  - scope
  - summary
  - source_session_id?
  - source_event_refs?
  - freshness?
  - metadata?
```

约束：

- durable memory 服务 recall，不服务 restore
- source refs 应足以支持来源追溯或语义等价能力

## Durable Memory Injection Objects

用于表达 file-backed durable memory injection 的最小共享对象。

```text
DurableMemoryInjectionSource
  - source_id
  - kind: agents_file
  - scope: user | project | subtree
  - file_path
  - applies_to
  - precedence
```

```text
LoadedMemoryInjection
  - source_ref
  - content
  - priority
  - applicable_target
```

约束：

- `DurableMemoryInjectionSource` 描述的是显式注入源，不等于 `DurableMemoryRecord`
- `LoadedMemoryInjection` 应进入 context injection plane，而不是 transcript plane
- `precedence` 必须足以表达 `~/.openagent/AGENTS.md -> workdir/AGENTS.md -> subtree/AGENTS.md` 的确定性顺序

## Agent And Gateway Relationship Objects

用于表达产品级实体关系与运行时绑定。

```text
AgentIdentity
  - agent_id
  - gateway_id
  - long_memory_id
```

```text
GatewayIdentity
  - gateway_id
  - agent_id
```

```text
ChannelAdapterInstance
  - channel_instance_id
  - channel_type
  - gateway_id
```

```text
ChatIdentity
  - chat_id
  - channel_instance_id
  - external_conversation_id
```

```text
ChatSessionBinding
  - chat_id
  - session_id
  - agent_id
  - gateway_id
  - channel_instance_id
```

约束：

- `1 Agent = 1 Gateway`
- `1 Chat = 1 Session`
- 一个 chat 只能归属一个 channel instance

## Session Ownership And Lease Objects

```text
HarnessInstance
  - harness_instance_id
  - agent_id
  - gateway_id
  - session_id?
  - runtime_state_ref?
  - status: starting | active | recovering | stopped | failed
```

```text
SessionHarnessLease
  - session_id
  - harness_instance_id
  - agent_id
  - acquired_at
  - lease_state: active | released | expired | recovering
  - resume_token?
```

```text
ShortTermMemoryRef
  - session_id
  - short_memory_id
```

```text
AgentLongTermMemoryRef
  - agent_id
  - memory_space_id
```

约束：

- `Gateway` 管理的是 `HarnessInstance`，不是 `AgentRuntime`
- `1 Session = 1 ShortTermMemoryRef`
- `1 Agent = 1 AgentLongTermMemoryRef`
- `SessionHarnessLease` 是单活 lease，不是并发 owner 集合
- `HarnessInstance` 是 worker/object identity；`AgentRuntime` 是其内部 turn loop
- `resume` 应恢复或转移 lease，而不是复制出第二个 active session writer

## MCP Session And Capability Objects

用于表达 MCP `2025-11-25` 协议接入的最小共享对象。

```text
McpSessionHandle
  - server_id
  - protocol_version
  - negotiated_capabilities
  - transport
  - session_id?
  - auth_state?
```

```text
McpServerCapabilitiesView
  - tools?
  - prompts?
  - resources?
  - logging?
  - completions?
  - tasks?
```

```text
McpClientCapabilities
  - roots?
  - sampling?
  - elicitation?
  - tasks?
  - experimental?
```

约束：

- 这些对象描述 capability negotiation 的结果，不等于本地 `ToolRegistry`
- capability 未协商成功时，不得在 runtime adaptation 层伪装为可调用能力

## MCP Descriptor Objects

```text
McpToolDescriptor
  - server_id
  - name
  - title?
  - description?
  - input_schema?
  - annotations?
  - icons?
```

```text
McpPromptDescriptor
  - server_id
  - name
  - title?
  - description?
  - arguments?
```

```text
McpResourceDescriptor
  - server_id
  - uri
  - name
  - title?
  - description?
  - mime_type?
  - annotations?
  - size?
```

约束：

- `server_id + name` 或 `server_id + uri` 必须足以保留来源边界
- `McpResourceDescriptor.uri` 是资源身份锚点，不应在适配时丢失

## MCP Client Feature Objects

```text
McpRoot
  - uri
  - name?
  - writable?
```

```text
McpSamplingRequest
  - server_id
  - request_id
  - model_preferences?
  - messages
  - system_prompt?
  - tools?
  - tool_choice?
  - metadata?
```

```text
McpElicitationRequest
  - server_id
  - request_id
  - mode: form | url
  - title?
  - schema?
  - url?
  - explanation?
```

约束：

- `McpSamplingRequest` 与 [tools/mcp/client-capabilities.md](tools/mcp/client-capabilities.md) 中的 sampling bridge 语义一致
- `McpElicitationRequest` 应能稳定映射到 canonical `RequiresAction`

## MCP Utility Objects

```text
McpLogEvent
  - server_id
  - level
  - logger?
  - data
  - timestamp?
```

```text
McpTaskHandle
  - server_id
  - task_id
  - status
  - operation
  - result_ref?
  - metadata?
```

约束：

- `McpTaskHandle` 描述的是 MCP task lifecycle，不是本地 `ToolExecutionHandle`
- `McpLogEvent` 属于观测面，不应直接写入 transcript 作为 assistant message

## ImportedSkillManifest

用于表达导入后的 skill manifest。

推荐最小字段：

```text
ImportedSkillManifest
  - name
  - description
  - license?
  - compatibility?
  - metadata?
  - allowed_tools?
  - skill_root
  - skill_file
  - source
  - diagnostics[]
```

约束：

- `ImportedSkillManifest` 应同时兼容 Agent Skills 标准字段和 host-specific extension 分层
- `diagnostics` 应能表达 warning / error，而不是只靠日志侧输出

## SkillCatalogEntry

用于表达向模型或用户披露的 skill catalog 条目。

推荐最小字段：

```text
SkillCatalogEntry
  - name
  - description
  - location?
  - source
  - invocable_by_model
```

约束：

- catalog disclosure 应明显小于完整 activation disclosure
- 不应要求模型在 catalog 阶段看到完整 `SKILL.md`

## SkillActivationResult

用于表达 skill 被激活后注入当前 interaction 的结构化结果。

推荐最小字段：

```text
SkillActivationResult
  - skill_name
  - body
  - frontmatter_mode: full | stripped
  - skill_root
  - listed_resources[]
  - wrapped: boolean
  - activation_mode: model | user
```

约束：

- `SkillActivationResult` 应足以支持 compaction protection、dedupe 和 replay
- `listed_resources` 表示资源披露，不等于资源 eager loading

## ToolResult

用于统一 tool 或 tool-like action 的结果返回语义。

推荐最小字段：

```text
ToolResult
  - tool_name
  - success
  - tool_use_id?
  - content
  - structured_content?
  - error?
  - metadata?
  - persisted_ref?
  - truncated?
```

约束：

- 若 `success=false`，应优先通过 `error: AgentError` 表达失败原因
- tool 的 `progress/result/failed/cancelled` 过程语义应通过 `RuntimeEvent` 或 `ToolExecutionEvent` 表达，而不全部压缩进 `ToolResult`

## PersistedToolResultRef

用于表达工具大结果被外存化后的稳定引用。

推荐最小字段：

```text
PersistedToolResultRef
  - tool_use_id
  - storage_ref
  - preview?
  - original_size?
  - media_type?
  - created_at?
```

约束：

- `PersistedToolResultRef` 必须稳定到同一 `tool_use_id`
- compact、resume、branch restore 后，不应要求重新生成原始结果才能恢复该引用
- `storage_ref` 可以是文件路径、对象存储键或等价句柄，但必须可序列化

## SecurityBoundary

用于统一表达 execution / environment sandbox 对能力边界的投影。

推荐最小字段：

```text
SecurityBoundary
  - filesystem
  - network
  - process
  - environment
  - device_access?
  - host_access?
  - escalation_policy?
```

约束：

- `SecurityBoundary` 必须是可序列化的结构化对象，不能只存在于 UI 文案中
- `Local` 与 `Cloud` 可以用不同执行技术，但应投影到同一能力维度

## CapabilityView

用于表达某一时刻 runtime 可暴露给模型或上层系统的能力视图。

推荐最小字段：

```text
CapabilityView
  - tools
  - skills
  - commands?
  - mcp_servers?
  - hands?
  - policies?
  - hosting_profile?
```

## HostingProfile

用于表达模块部署位置与职责分布，不改变五大模块边界。

推荐最小字段：

```text
HostingProfile
  - profile_type: local | cloud | hybrid
  - harness_location
  - session_location
  - tool_execution_location
  - sandbox_location
  - orchestration_location
```

约束：

- `HostingProfile` 影响部署映射，不影响模块语义
- 对本规范而言，默认 profile 是 `local`

## TaskRecord

用于表达 task-driven runtime 中的统一任务对象。

推荐最小字段：

```text
TaskRecord
  - task_id
  - type
  - status
  - description
  - session_id?
  - agent_id?
  - parent_task_id?
  - output_ref?
  - start_time
  - end_time?
  - terminal_state?
  - metadata?
```

推荐 `status` 至少与 task event 语义兼容：

- `pending`
- `running`
- `completed`
- `failed`
- `cancelled`
- `killed`

## TaskEvent

用于表达 task 生命周期中的增量事件。

```text
TaskEvent
  - task_id
  - event_id
  - timestamp
  - type: started | progress | notification | completed | failed | cancelled | killed
  - payload
  - terminal_state?
  - error?
```

## ContextModifierCommit

用于表达工具执行产生的上下文修改在何时提交。

```text
ContextModifierCommit
  - execution_id
  - tool_use_id
  - stage: staged | committed | discarded
  - order
  - modifier_ref?
```

## SessionCheckpoint

用于表达 session durable append 的提交位置。

```text
SessionCheckpoint
  - session_id
  - last_event_id
  - cursor
  - committed_at
```

## 规范结论

- `RuntimeEvent`、`TerminalState`、`AgentError`、`RequiresAction`、`ToolResult`、`CapabilityView`、`TaskRecord`、`TaskEvent`、`SessionCheckpoint` 应作为跨语言共享对象保留
- `PersistedToolResultRef` 与 `ContextModifierCommit` 也应作为跨语言共享对象保留
- 这些对象的语义必须独立于具体文件组织和类型系统
- `object-model.md` 应成为各子模块引用的 canonical field table，而不是各自发明局部变体
