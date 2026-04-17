# Schema And Serialization

## 目标

本文件说明标准对象如何做跨语言序列化、版本兼容和字段演进。

## 设计原则

- 先稳定对象语义，再稳定线格式
- 向后兼容优先于字段美观
- 新字段默认优先做可选字段
- 破坏性变更必须显式声明版本边界

## 字段演进规则

- 新增字段优先为可选
- 删除字段前必须经历弃用期
- 改变字段含义视为破坏性变更
- 仅改变字段名但不保留兼容映射，也视为破坏性变更

## 序列化要求

- 所有标准对象必须可映射到稳定的 JSON-like 结构
- 时间字段应使用明确的时间戳语义
- 枚举字段应使用稳定字符串语义
- 未识别字段应允许保留或透传，不能默认报错导致系统无法升级

## 版本声明

推荐每个可交换对象至少支持：

```text
SchemaEnvelope
  - schema_name
  - schema_version
  - payload
```

推荐优先为这些 canonical objects 提供稳定 envelope：

- `RuntimeEvent`
- `TerminalState`
- `AgentError`
- `RequiresAction`
- `PolicyDecision`
- `TaskEvent`
- `SessionCheckpoint`
- `ResumeSnapshot`
- `DurableMemoryRecord`
- `DurableMemoryInjectionSource`
- `LoadedMemoryInjection`
- `ToolResult`
- `PersistedToolResultRef`
- `ImportedSkillManifest`
- `SkillCatalogEntry`
- `SkillActivationResult`
- `McpSessionHandle`
- `McpServerCapabilitiesView`
- `McpClientCapabilities`
- `McpToolDescriptor`
- `McpPromptDescriptor`
- `McpResourceDescriptor`
- `McpRoot`
- `McpSamplingRequest`
- `McpElicitationRequest`
- `McpLogEvent`
- `McpTaskHandle`

对应 canonical 字段表见 [object-model.md](object-model.md)。

## Canonical Object Guidance

对于本规范中新增且会跨模块、跨语言流转的对象，推荐遵循以下序列化约束：

- `PolicyDecision`
  - `decision` 使用稳定字符串枚举
  - `passthrough` 仅允许内部链路；若进入可交换对象，应先归约为最终决策
- `RequiresAction`
  - `request_id`、`tool_use_id`、`action_ref` 应保持稳定引用语义
  - 便于 session resume / approval replay 绑定原阻塞点
- `ResumeSnapshot`
  - `checkpoint`、`runtime_state`、`working_state` 应序列化为可恢复对象，而不是仅文本摘要
- `DurableMemoryRecord`
  - `source_event_refs`、`scope`、`freshness` 应为稳定字段
  - 不应依赖实现私有路径语义才能理解
- `DurableMemoryInjectionSource`
  - `kind`、`scope`、`precedence` 使用稳定字符串或稳定有序字段
  - `file_path` 与 `applies_to` 应保持宿主可解释但不依赖 transcript 的语义
- `LoadedMemoryInjection`
  - `source_ref` 必须能稳定追溯到原始 injection source
  - 不应被序列化成 transcript message 或 assistant/user role message
- `PersistedToolResultRef`
  - `storage_ref` 必须可序列化
  - compact / resume / branch restore 后，`tool_use_id -> storage_ref` 映射不得漂移
- `ImportedSkillManifest`
  - Agent Skills 标准字段与 host-specific extension 应可分层序列化
  - `diagnostics` 应允许结构化透传
- `SkillCatalogEntry`
  - 应适合低成本 disclosure，不应隐式扩展为完整 skill body
- `SkillActivationResult`
  - `frontmatter_mode`、`activation_mode` 使用稳定字符串枚举
  - `listed_resources` 与完整资源内容分离
- `McpSessionHandle`
  - `protocol_version`、`transport` 使用稳定字符串语义
  - `negotiated_capabilities` 应保持对象分层，而不是压成单个自由文本字段
- `McpResourceDescriptor`
  - `uri` 必须按稳定 URI 语义序列化
  - 不应依赖宿主私有路径重写才能被理解
- `McpSamplingRequest`
  - `tools`、`tool_choice` 仅在 capability 协商允许时出现
- `McpElicitationRequest`
  - `mode` 使用稳定字符串枚举
  - `schema` 与 `url` 应保持互斥或最小重叠语义
- `McpTaskHandle`
  - `status` 与 deferred result reference 应保持稳定字段，不应仅用自由文本表达

## Envelope Recommendations

建议对跨 host / 跨进程交换对象统一采用：

```text
SchemaEnvelope
  - schema_name
  - schema_version
  - payload
  - metadata?
```

其中 `metadata` 推荐用于：

- artifact source
- host profile
- feature flags that materially affect semantics
- backwards-compatible optional annotations

## Time And Identifier Semantics

推荐约束：

- 时间字段使用明确时间戳格式，不依赖本地时区文本
- ID 字段保持宿主无关的稳定字符串语义
- `tool_use_id`、`request_id`、`session_id`、`task_id` 不应在跨语言适配时被重编码为仅本地可识别的结构

## Unknown Field Handling

对于 forward-compatible 升级，推荐：

- reader 保留未知字段
- writer 不主动剥离来自上游 envelope 的未知字段，除非存在安全风险
- canonical object 的核心字段缺失时，才视为 schema violation

## 兼容策略

- reader 应尽量向前兼容新增可选字段
- writer 应避免无意义写出实验字段
- major 版本用于破坏性对象语义变化
- minor 版本用于新增可选字段和可兼容行为

## 规范结论

- 标准对象必须有清晰的版本演进规则
- 跨语言兼容首先依赖稳定序列化语义，而不是同名类型定义
- 对象版本演进应优先围绕 canonical object model 展开，而不是围绕某个单一实现仓库的本地类型
- `PolicyDecision`、`ResumeSnapshot`、`DurableMemoryRecord`、`PersistedToolResultRef` 这类跨模块对象应优先纳入稳定 envelope
- `DurableMemoryInjectionSource`、`LoadedMemoryInjection` 这类 file-backed memory injection 对象也应优先纳入稳定 envelope
- `ImportedSkillManifest`、`SkillCatalogEntry`、`SkillActivationResult` 这类 skills 生命周期对象也应优先纳入稳定 envelope
- MCP lifecycle / capability / descriptor 对象也应纳入稳定 envelope，避免 transport-specific 本地类型泄漏到跨语言接口
