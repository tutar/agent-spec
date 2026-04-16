# Reflection And Verification Commands

## 职责

`Reflection And Verification Commands` 定义一类 command-like capability：对当前任务、已有结果或已收集证据再做一轮 review、critique、reflection 或 verification。

它们不是 Agent Skills 规范中的 `skill`，也不是普通 `ToolDefinition`。

它们的特点是：

- 对 harness 来说，它们像一个可触发的 `command`
- 触发后不一定在当前线程内直接执行
- 默认执行后端可以委托给 orchestration 拉起 subagent / task
- 结果会作为 verdict、findings、critique 或附加上下文回流当前链路

它至少覆盖两类 capability：

- `reflection command`
  对已有输出、证据或上下文做后置复盘与 critique
- `verification command`
  对主 agent 产物做独立复核，并产出结构化 verdict

规范上要明确：

- verification 不等于主 agent 自己再想一遍
- reflection / verification command 不等于 Agent Skills 生态对象
- command capability 的默认执行可以由 orchestration 承担，但 capability surface 归属 `Tools`

## 稳定接口

推荐最小接口：

```text
ReviewCommand
  - kind: reflection | critique | review | verification
  - description
  - required_inputs
  - allowed_tools?
  - execution_mode?
  - invoke(context) -> ReviewResult

ReviewContext
  - target_session
  - target_agent?
  - original_task
  - changed_artifacts?
  - evidence_scope?
  - review_policy?

ReviewResult
  - kind
  - verdict: pass | fail | partial
  - evidence[]
  - findings[]
  - limitations[]
  - output_ref?
```

其中：

- `kind`
  表示 command 所暴露的 review 能力语义
- `execution_mode`
  可表达本地 prompt path、同步 verifier、后台 task 等实现形态
- `invoke()`
  代表 capability surface 上的统一触发动作，不要求必须由本地函数直接实现

## 默认实现

当前代码库的默认实现重点在 verifier agent：

- [tools/AgentTool/built-in/verificationAgent.ts](../../cc/tools/AgentTool/built-in/verificationAgent.ts)
  内置 verification specialist，要求独立运行命令、拿到证据、输出 `VERDICT`
- [tools/AgentTool/builtInAgents.ts](../../cc/tools/AgentTool/builtInAgents.ts)
  注册 built-in verifier
- [commands/init-verifiers.ts](../../cc/commands/init-verifiers.ts)
  初始化项目级 verifier command / prompt 能力
- [tools/TodoWriteTool/TodoWriteTool.ts](../../cc/tools/TodoWriteTool/TodoWriteTool.ts)
  在任务收尾阶段推动 verifier 进入标准流程
- [tools/TaskUpdateTool/TaskUpdateTool.ts](../../cc/tools/TaskUpdateTool/TaskUpdateTool.ts)
  在多任务收尾时要求验证步骤

当前默认实现表明：

- 对上层 harness / agent 来说，verification 是一个可触发 capability
- 默认后端通常通过独立 agent 完成
- verifier 的 prompt、工具边界和输出格式独立于主 agent
- capability surface 与执行后端不是同一层对象

## 要解决的问题

- 如何把 review / reflection / verification 作为可发现、可触发的 command-like capability 暴露出来
- 如何避免把这类 capability 误归类为 Agent Skills
- 如何表达“上层像 command，下层靠 subagent/task 执行”这类双层语义
- 如何为非平凡任务提供独立 verdict，同时不把 verification 降格为普通 tool call
- 如何区分 correctness verification、self-check、memory consolidation 和 compact

## 规范结论

- `reflection / verification` 应首先被建模成 `Tools` 域内的 command-like capability
- 它们不属于 `tools/skills`，因为 `tools/skills` 只服务 Agent Skills 生态
- 它们的默认执行后端可以是 `Orchestration` 中的 verifier task / subagent
- verification 输出应结构化，至少包含 verdict 与 evidence
- 该文档定义 capability surface；执行生命周期见 [../orchestration/agent-orchestration/evaluation-and-verification.md](../orchestration/agent-orchestration/evaluation-and-verification.md)
