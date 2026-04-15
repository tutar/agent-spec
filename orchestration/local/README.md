# Local Hosting Profile

## 目标

`Local Hosting Profile` 描述单机部署 agent 系统的标准职责分布。

这里的重点不是交互方式，而是部署方式：

- channel、gateway、harness、session、tools、sandbox、orchestration 可以部署在同一台机器上
- 模块之间可以 direct-call
- session、task、verifier、memory 默认以本地持久化和本地执行为主

## 角色分布

在 Local 场景下，推荐的职责分布是：

- `ChannelAdapter`
  本地接入壳，例如 terminal、GUI、embedded client
- `Gateway`
  本地输入标准化、session binding、control routing
- `Harness`
  本地 turn evaluation
- `Session`
  本地 durable transcript + restore
- `Orchestration`
  本地 task / subagent / verifier / background task
- `Sandbox`
  以 `Execution Sandbox` 为主

若需要环境级边界，Local 侧应优先表现为给定目录范围内的访问权，而不是远端 execution substrate。详见 [../../sandbox/local/environment-sandbox.md](../../sandbox/local/environment-sandbox.md)。

## 典型特征

- session 往往本地持久化
- permission / requires_action 通常可由本机宿主直接完成
- background task 与 subagent 常由本机进程承载
- verifier 常作为本地独立 agent/task 执行
- direct-call contract 是默认优化路径
- 应用或系统重启后，不应破坏已 durable session 的 resume 能力

这里的“本地”只描述常见部署优化，不代表接口可以假设所有组件与 harness 共进程。
相关约束见 [../../harness/deployment-boundaries.md](../../harness/deployment-boundaries.md)。

## 对 orchestration 的要求

- 支持本地 task-first orchestration
- 支持 foreground / background agent 切换
- 支持本地 verifier 作为标准后置步骤
- 支持本地 permission / requires_action 流
- 支持本地 session resume

## 默认实现映射

当前代码库中的默认 Local hosting profile 主要映射到：

- [screens/REPL.tsx](../../../cc/screens/REPL.tsx)
- [QueryEngine.ts](../../../cc/QueryEngine.ts)
- [utils/sessionStorage.ts](../../../cc/utils/sessionStorage.ts)
- [tasks/LocalAgentTask/LocalAgentTask.tsx](../../../cc/tasks/LocalAgentTask/LocalAgentTask.tsx)
- [tools/AgentTool/built-in/verificationAgent.ts](../../../cc/tools/AgentTool/built-in/verificationAgent.ts)

## 规范结论

- Local profile 应被视为单机部署形态，而不是某种特定交互壳
- 在该 profile 中，orchestration 通常以本地 task、subagent、permission 交互为中心
