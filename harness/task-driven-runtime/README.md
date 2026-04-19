# Task-Driven Runtime

本目录收拢 `Harness` 域内与本地 task-driven runtime 直接相关的语义。

它回答四类问题：

- runtime task 如何被托管、恢复和通知
- background agent 如何进入标准 harness 链路
- verification 如何作为本地运行时单元执行
- 多 worker 协作中的 work allocation 如何与 runtime task 分层

这组文档描述的是 `Local-first` 默认运行时主线，不等于 cloud deployment 下的托管控制面。
cloud 侧的控制面语义见 [../../orchestration/README.md](../../orchestration/README.md) 与 [../../orchestration/cloud/README.md](../../orchestration/cloud/README.md)。

## 目录内文档

- [task-manager.md](task-manager.md)
- [background-agent.md](background-agent.md)
- [evaluation-and-verification.md](evaluation-and-verification.md)
- [work-allocation.md](work-allocation.md)
