# Conformance

## 目标

本文件定义各语言 agent 实现的一致性要求。

不同语言实现可以使用不同的并发模型、序列化库、HTTP 客户端和类型系统，但不能改变 agent 的核心行为语义。

本规范采用两层合规模型：

- `核心合规`
  实现某个模块时必须满足的最小行为语义
- `扩展合规`
  对 prompt cache、memory consolidation、background agent 等标准扩展的行为一致性要求

一个实现可以只实现核心层；只有在对外宣称实现某个标准扩展时，才需要通过对应扩展用例。

## 一致性范围

需要跨语言保持一致的内容：

- 事件语义
- 终止状态语义
- tool 权限语义
- requires_action 语义
- context governance 触发语义
- 模型能力路由语义
- task 生命周期语义
- Skills / MCP / MCPB 兼容语义
- `tool / skill / command / mcp` 的角色语义
- `Local-first` 默认语义下的 `Cloud-compatible` 边界
- persisted tool result reference 语义
- policy decision auditability 语义
- Agent Skills import / disclosure / activation 语义
- MCP lifecycle / transport / capability negotiation 语义

允许不同实现存在差异的内容：

- 内部类名、函数名、文件组织
- 线程、协程、异步任务实现方式
- 本地缓存、序列化、日志框架
- 网络重试的具体实现细节

## 强制一致的行为

### 1. Runtime Event Semantics

各语言实现必须对外暴露等价的事件分类。

例如：

- `assistant_delta`
- `tool_started`
- `tool_progress`
- `tool_result`
- `requires_action`
- `turn_completed`
- `turn_failed`

事件字段名可以按语言习惯调整，但事件含义不能变化。

### 2. Terminal State Semantics

各语言实现必须能区分：

- 正常完成
- 用户中止
- 权限阻塞
- 预算终止
- 运行失败

不能把这些终止类型混成单一 `error`。

### 3. Tool Execution Semantics

各语言实现必须保持：

- 并发安全工具可并发
- 非并发安全工具必须串行
- context mutation 不能并发乱序应用
- 用户中断行为必须尊重 tool 的中断语义

### 4. Policy Semantics

各语言实现必须保持：

- 权限决策有 `allow/deny/ask`
- `requires_action` 是结构化状态
- 非交互环境可自动拒绝
- 静态规则与运行时审查是两层逻辑

### 5. Context Governance Semantics

各语言实现必须保持：

- 超长上下文会触发治理
- 超长工具结果不能无约束进入上下文
- compact 与 overflow recovery 可被 runtime 调用
- budget continuation 有一致的停止条件

若实现声明支持 prompt cache 或高级 compact 策略，还必须保持：

- stable prefix 语义一致
- dynamic suffix 语义一致
- fork sharing 语义一致
- cache break detection 语义一致
- compact boundary 后的恢复语义一致

### 6. Ecosystem Compatibility Semantics

各语言实现若宣称实现 `Tools` 模块，应保持以下兼容语义：

- skill discovery / load / activate 的基本行为一致
- MCP tool / resource / prompt 的接入语义一致
- MCP lifecycle、transport、auth、server/client capabilities 的核心语义一致
- 桌面端实现的 bundle install / load / verify 语义一致
- `skill` 与 `tool` 是不同稳定接口
- `command` 是共享对象模型，不是第六顶层模块
- `mcp` 是协议接入层，不等于一组远端 tools

允许实现差异：

- 具体缓存方式
- 传输实现
- UI 呈现形式

不允许差异：

- skill 与 tool 的能力来源模型
- command 与 skill/tool 的角色分层
- MCP prompt/resource/tool 的基本角色定义
- MCP core 与 host extension 的边界
- MCP skill 与 MCP prompt 的区分语义
- 桌面 bundle 的安装后生命周期语义

若实现声明支持 Agent Skills specification 导入，还必须保持：

- `SKILL.md` 目录导入语义一致
- skill precedence / shadowing 语义 deterministic
- catalog disclosure 与 activation disclosure 分层一致
- skill activation 结果具备可 dedupe / 可 compaction-protect 的稳定语义

若实现声明支持 MCP `2025-11-25` compatibility，还必须保持：

- `initialize -> initialized` 生命周期闭合
- version / capability negotiation 结果稳定
- `stdio` 与 `Streamable HTTP` 的能力语义等价
- `roots / sampling / elicitation` 仅在 capability 协商成功时对外生效
- `tools / prompts / resources` 的 pagination 与 list-changed 语义一致
- `resources/subscribe` 与 `notifications/resources/updated` 的观察语义一致
- `mcp skill` 等 host extension 不被误当成 core conformance requirement

### 7. Session Resume And Memory Semantics

各语言实现必须保持：

- append-only transcript 或语义等价 durable event log
- `SessionCheckpoint` 的 durable progress 含义
- `RequiresAction` 可恢复语义
- harness / sandbox 替换后仍可基于 session 继续
- `1 Chat = 1 Session` 的绑定语义稳定
- `1 Session = 1 Short-Term Memory` 的绑定语义稳定

若实现声明支持 session memory 或 memory consolidation，还必须保持：

- short-term session continuity 与 transcript 分离
- durable memory recall 不承担 restore 责任
- consolidation 不得破坏 resume 语义
- branch / sidechain transcript 与父 session 的追溯关系一致
- 同一 agent 下多个 session 可共享同一个 global long-term memory
- 同一 session 任一时刻只能有一个 active harness lease

若实现声明支持 `AGENTS.md` file-backed memory injection，还必须保持：

- `~/.openagent/AGENTS.md -> workdir/AGENTS.md -> subtree/AGENTS.md` 的优先级顺序 deterministic
- 注入结果进入 context plane，而不是 transcript / event log
- 子目录 `AGENTS.md` 只影响对应子树
- compact / resume 后不通过 transcript 反推 `AGENTS.md`

### 8. Tool Execution And Persistence Semantics

各语言实现必须保持：

- `ToolDefinition` 的校验失败、权限失败、执行失败三类语义可区分
- batch executor 与 streaming executor 的 terminal state / error class 语义一致
- context mutation 的 staged / committed / discarded 语义一致

若实现声明支持大结果外存化，还必须保持：

- `PersistedToolResultRef` 可序列化
- resume / compact / branch restore 后引用仍稳定
- preview 与 full result reference 的关系一致

### 9. Policy Decision Semantics

各语言实现必须保持：

- `PolicyDecision` 可序列化、可审计
- `allow / deny / ask` 的最终结果语义一致
- `ask` 与 `requires_action` 分层一致
- background / async session 不会生成不可达的 approval dead-end

## 一致性测试基线

每种语言的 agent 实现至少应具备以下测试类型。

### Contract Tests

验证标准输入输出契约：

- tool definition 契约
- model capability view 契约
- requires_action 契约
- runtime terminal state 契约
- session checkpoint / resume snapshot 契约
- agent / gateway / chat / session binding 契约
- security boundary 契约
- persisted tool result reference 契约
- context modifier commit 契约
- policy decision 契约
- durable memory record 契约
- durable memory injection source 契约
- imported skill manifest 契约
- skill catalog entry 契约
- skill activation result 契约
- MCP session handle / capability view 契约
- MCP descriptor / request object 契约

### Behavior Tests

验证行为一致性：

- 并发 tool 执行顺序
- 权限 ask/deny/allow 分支
- compact 触发与恢复
- 子 agent 生命周期
- wake / resume 后的 working state 延续
- chat-session binding 与 single active harness lease 一致性
- prompt cache 扩展的命中稳定性
- short-term memory 更新与 recall linkage
- persisted tool result 在 compact / resume 后的稳定性
- batch / streaming executor 的终态一致性
- allow / deny / ask 到 requires_action / deny error 的映射一致性
- skills discovery precedence 与 activation disclosure 一致性
- MCP lifecycle / pagination / subscription / client-feature 行为一致性
- `AGENTS.md` loading precedence 与 subtree applicability 一致性

### Failure Tests

验证异常情况下的统一行为：

- 模型限流
- prompt too long
- tool 执行失败
- 用户中断
- 非交互环境拒绝审批
- sandbox violation
- compact / cache 恢复失败后的终态收敛
- harness restart 后 short-term memory 不丢失
- tool result persistence failure 的回退语义
- consolidation failure / lock conflict 的安全退化语义
- malformed skill import 的安全降级语义
- MCP auth discovery / scope upgrade / transport failure 的统一退化语义
- 缺失或冲突的 `AGENTS.md` source 不得破坏 session restore

### Compatibility Tests

验证不同模型接入后语义不漂移：

- 支持原生 tool calling 的模型
- 不支持原生 tool calling 的模型
- 支持 structured output 的模型
- 不支持 structured output 的模型

还应包含生态兼容测试：

- Agent Skills skill 目录能被发现并激活
- MCP server 的 tools/resources/prompts 能被正确枚举与调用
- MCP server/client capabilities 在声明支持时可被正确协商与投影
- 桌面端 `.mcpb` bundle 能被安装、校验与加载
- 使用内部 command model 的实现，不会把 skill 或 mcp prompt 错误暴露成 tool
- `AGENTS.md` 可作为 file-backed durable memory injection 被发现并按优先级注入
- 一个 gateway 可管理多个 channel/chat/session，但只归属于一个 agent

## 推荐测试工件

建议维护一套跨语言共享的测试工件：

- 标准事件录制样本
- 标准 tool 用例集
- 标准权限决策样本
- 标准 compact 场景样本
- 标准模型能力矩阵样本

这些工件应脱离具体语言实现，作为所有 agent 的共同验收基线。

规范目录中的共享样例入口见：

- [conformance/README.md](conformance/README.md)
- [conformance/cases/basic-turn.md](conformance/cases/basic-turn.md)
- [conformance/cases/tool-call-roundtrip.md](conformance/cases/tool-call-roundtrip.md)
- [conformance/cases/requires-action-approval.md](conformance/cases/requires-action-approval.md)
- [conformance/cases/session-resume.md](conformance/cases/session-resume.md)
- [conformance/cases/background-agent.md](conformance/cases/background-agent.md)
- [conformance/cases/hosting-profile-equivalence.md](conformance/cases/hosting-profile-equivalence.md)
- [conformance/cases/mcp-tool-adaptation.md](conformance/cases/mcp-tool-adaptation.md)
- [conformance/cases/mcp-initialize-and-version-negotiation.md](conformance/cases/mcp-initialize-and-version-negotiation.md)
- [conformance/cases/mcp-streamable-http-post-and-sse.md](conformance/cases/mcp-streamable-http-post-and-sse.md)
- [conformance/cases/mcp-roots-list-and-list-changed.md](conformance/cases/mcp-roots-list-and-list-changed.md)
- [conformance/cases/mcp-sampling-with-tools.md](conformance/cases/mcp-sampling-with-tools.md)
- [conformance/cases/mcp-elicitation-form-vs-url.md](conformance/cases/mcp-elicitation-form-vs-url.md)
- [conformance/cases/mcp-resource-subscribe-and-list-changed.md](conformance/cases/mcp-resource-subscribe-and-list-changed.md)
- [conformance/cases/mcp-prompt-pagination-and-get.md](conformance/cases/mcp-prompt-pagination-and-get.md)
- [conformance/cases/mcp-tool-pagination-and-call.md](conformance/cases/mcp-tool-pagination-and-call.md)
- [conformance/cases/mcp-auth-discovery-and-scope-upgrade.md](conformance/cases/mcp-auth-discovery-and-scope-upgrade.md)
- [conformance/cases/mcp-host-extension-skill-discovery.md](conformance/cases/mcp-host-extension-skill-discovery.md)
- [conformance/cases/sandbox-deny.md](conformance/cases/sandbox-deny.md)
- [conformance/cases/memory-recall-and-consolidation.md](conformance/cases/memory-recall-and-consolidation.md)
- [conformance/cases/cloud-wake-and-reprovision.md](conformance/cases/cloud-wake-and-reprovision.md)
- [conformance/cases/prompt-cache-stable-prefix.md](conformance/cases/prompt-cache-stable-prefix.md)
- [conformance/cases/prompt-cache-dynamic-suffix.md](conformance/cases/prompt-cache-dynamic-suffix.md)
- [conformance/cases/prompt-cache-fork-sharing.md](conformance/cases/prompt-cache-fork-sharing.md)
- [conformance/cases/prompt-cache-break-detection.md](conformance/cases/prompt-cache-break-detection.md)
- [conformance/cases/prompt-cache-strategy-equivalence.md](conformance/cases/prompt-cache-strategy-equivalence.md)

## 版本兼容要求

规范升级时应明确：

- 哪些字段是新增可选字段
- 哪些行为是向后兼容变更
- 哪些行为变化属于破坏性变更

各语言 agent 实现应对外声明自己实现的规范版本。

## 最低合规要求

一个语言 agent 实现若要宣称“实现了本规范”，至少必须具备：

- 标准化模型适配层
- 标准化 runtime 事件流
- 标准化 session checkpoint / resume 语义
- 标准化 tool 契约
- 标准化 policy 决策
- 标准化 context governance 基线

若要宣称“实现了本规范的 `Tools` 模块”，至少还应具备：

- Agent Skills compatibility
- MCP compatibility

若目标是桌面端，还应具备：

- MCPB compatibility

若要宣称实现以下标准扩展，还应分别通过对应用例：

- `Harness.PromptCacheStrategy`
  通过 prompt cache 相关 golden cases
- `Harness.Streaming`
  通过 delta / tool streaming / recovery 用例
- `Harness.TaskDrivenRuntime`
  通过 background agent 用例
- `Session.MemoryConsolidation`
  通过 memory recall / consolidation 用例
- `Orchestration.ManagedControlPlane`
  通过 wake-and-reprovision 用例

缺少其中任一项，只能称为局部实现。

## 规范结论

- 规范的目标是行为一致，不是代码结构一致
- 各语言 agent 实现可以自由实现，但不能自由改变语义
- 合规测试应与语言实现解耦，作为共享验收标准存在
- 核心合规与扩展合规应分开声明，避免把所有高级优化都误当成最低门槛
