# Task

本目录收拢 `Harness` 域内与 task 子系统直接相关的语义。

它回答几类问题：

- task 是什么、注册到哪里、如何成为单一事实来源
- task 的生命周期、输出 cursor、通知去重、恢复与回收如何统一
- background agent 如何进入标准 harness 链路
- verification 如何作为本地运行时单元执行
- 多 worker 协作中的 work allocation 如何与 task 分层

这里的 task 是独立执行层对象，不等于 `AgentRuntime` 的 turn state machine。
`AgentRuntime` 负责一轮 agent turn 的推进；task 负责长生命周期执行对象的注册、状态、输出、通知、恢复和回收。
多 agent 的委派、teammate 执行和消息路由见 [../multi-agent/README.md](../multi-agent/README.md)。

这组文档描述的是 `Local-first` 默认 task 主线，不等于 cloud deployment 下的托管控制面。
cloud 侧的控制面语义见 [../../orchestration/README.md](../../orchestration/README.md) 与 [../../orchestration/cloud/README.md](../../orchestration/cloud/README.md)。

## 目录内文档

- [task-model.md](task-model.md)
- [task-lifecycle.md](task-lifecycle.md)
- [task-manager.md](task-manager.md)
- [background-agent.md](background-agent.md)
- [verification.md](verification.md)
- [work-allocation.md](work-allocation.md)
