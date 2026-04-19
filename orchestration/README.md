# Orchestration

## 职责

`Orchestration` 负责 cloud 托管 agent 的控制面。

模块总览见 [../module-overview.md](../module-overview.md)，术语归属见 [../terminology-and-ownership.md](../terminology-and-ownership.md)。

它负责：

- 管理 many brains / many hands
- 实现 lazy provisioning、wake-based recovery 和 remote hand routing
- 连接 harness、session、sandbox 与远端 execution targets
- 表达 cloud hosting profile 下的职责分布

## 稳定接口

推荐最小接口：

```text
Orchestrator
  - provision_hand()
  - wake_session()
  - route_execution()
  - recover_managed_agent()
  - terminate_managed_agent()
```

## 要解决的问题

- 如何让一个系统同时拥有 many brains 与 many hands
- 如何在 hand 或 harness 失效后恢复工作
- 如何在不牺牲 TTFT 的前提下实现按需 provision
- 如何在 cloud 部署下保持 session、harness、hand 解耦后的恢复能力

本目录只收拢 cloud 托管控制面语义。
local 模式下的 task、background task、verification、teammate 和消息路由见 [../harness/task/README.md](../harness/task/README.md) 与 [../harness/multi-agent/README.md](../harness/multi-agent/README.md)。

## 目录内文档

- [cloud/README.md](cloud/README.md)
- [cloud/managed-orchestration.md](cloud/managed-orchestration.md)
- [conformance-scenarios.md](conformance-scenarios.md)

## 规范结论

- orchestration 是系统控制面，不是某个具体 task 类型
- orchestration 仅负责 cloud 托管控制面，不承载 local task-driven runtime
- orchestration 必须显式表达 many brains / many hands / wake / provision 语义
