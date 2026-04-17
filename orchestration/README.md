# Orchestration

## 职责

`Orchestration` 负责连接 harness、session、tools 和 sandbox，把它们组织成一个可运行、可恢复、可扩展的 managed-agent 系统。

模块总览见 [../module-overview.md](../module-overview.md)，术语归属见 [../terminology-and-ownership.md](../terminology-and-ownership.md)。

它负责：

- 管理 task 生命周期
- 管理协作任务分配
- 管理 subagent / background / remote agent
- 管理 many brains / many hands
- 实现 lazy provisioning、wake-based recovery 和任务分配

在当前规范中，agent 编排不应只被表达为一个抽象的 `spawn_agent()`，而应至少区分几种标准模式：

- synchronous worker
- background agent task
- persistent teammate
- remote agent task

## 稳定接口

推荐最小接口：

```text
Orchestrator
  - spawn_agent()
  - send_input()
  - background_agent()
  - terminate_agent()
  - await_agent()
  - register_task()
  - resume_task()
  - assign_hand()
  - recover_from_failure()
```

## 默认实现

当前代码库中的默认 orchestration 实现是 task-first orchestration：

  汇总当前支持的 task types
  承担本地子 agent 生命周期
  承担长期存活 teammate 生命周期
  承担远端 agent 生命周期
  作为 agent spawn routing 入口
  作为默认本地 agent loop 执行内核

## 要解决的问题

- 如何统一本地、后台、远端和子代理生命周期
- 如何区分一次性 worker、后台 agent、长期 teammate、远端 agent
- 如何让一个系统同时拥有 many brains 与 many hands
- 如何在 hand 或 harness 失效后恢复工作
- 如何把任务执行与单次 tool 调用分离
- 如何把独立 verifier 纳入标准编排链

## 目录内文档

- [agent-orchestration/README.md](agent-orchestration/README.md)
- [local/README.md](local/README.md)
- [cloud/README.md](cloud/README.md)
- [cloud/managed-orchestration.md](cloud/managed-orchestration.md)

## 规范结论

- orchestration 是系统控制面，不是某个具体 task 类型
- orchestration 必须显式表达 agent 编排模式，而不是只保留一个模糊 spawn 概念
- 默认实现可以从 task-first 开始，但接口必须支持 many brains / many hands
