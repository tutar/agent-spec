# Cloud Hosting Profile

## 目标

`Cloud Hosting Profile` 描述托管式 agent 系统的标准职责分布。

典型形态包括：

- Claude-style managed agents
- hosted control plane + remote execution
- many brains / many hands

## 角色分布

在 Cloud 场景下，推荐的职责分布是：

- `ChannelAdapter`
  API、chat integrations、web app、webhook、remote control
- `Gateway`
  多渠道 input normalization、session binding、control routing
- `Harness`
  可横向扩展的 turn runner
- `Session`
  durable remote event log / restore store
- `Orchestration`
  managed-agent control plane / many brains / many hands / lazy provisioning / wake-resume
- `Sandbox`
  以 `Environment Sandbox` 为主，`Execution Sandbox` 为辅

Cloud 侧的 `Environment Sandbox` 应被视为完整 execution substrate，而不只是 remote workspace。详见 [../../sandbox/cloud/environment-sandbox.md](../../sandbox/cloud/environment-sandbox.md)。

## 典型特征

- harness 与 execution target 往往分离部署
- session 与 runtime 进程不共生
- wake / resume / reprovision 是主路径能力
- 远端 sandbox、VPC、proxy、credential boundary 是核心问题
- harness 与 sandbox 应允许无状态替换，而不影响既有 session / task 的恢复

## 对 orchestration 的要求

- 支持 many brains / many hands
- 支持 lazy provisioning
- 支持 remote task / remote agent / remote verifier
- 支持 harness 崩溃后基于 session log 恢复
- 支持 execution target 失效后重 provision

## 与 managed orchestration 的关系

`Managed Orchestration` 是 Cloud profile 的核心实现风格，但 Cloud profile 额外强调：

- hosting boundary
- deployment boundary
- remote session / remote sandbox / proxy boundary

Cloud profile 也应被视为 agent 设计的约束来源，而不只是一个额外适配层。
换句话说，稳定接口必须先满足 cloud/distributed semantics，再允许 Local profile 提供 direct-call 优化。
相关约束见 [../../harness/deployment-boundaries.md](../../harness/deployment-boundaries.md)。

## 默认实现映射

当前代码库对这一 profile 的直接实现并不完整，但已有相关映射：

- [managed-orchestration.md](managed-orchestration.md)

## 规范结论

- Cloud profile 应被视为默认的托管/分布式宿主形态
- 在该 profile 中，orchestration 必须显式承担 remote provisioning、wake、resume、recovery
