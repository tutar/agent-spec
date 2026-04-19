# Multi-Agent

本目录收拢 `Harness` 域内与 local multi-agent coordination 直接相关的语义。

它回答的问题是：

- 多 agent 在本地语义下有哪些执行形态
- subagent 与 teammate 的区别是什么
- leader 如何委派 worker，worker 如何回流结果
- 多 agent 之间有哪些消息路由通道
- transcript / viewed transcript / mailbox / task output 分别属于哪一层

这里的 multi-agent 不是 cloud control plane。
cloud 侧的 many brains / many hands 见 [../../orchestration/README.md](../../orchestration/README.md)。

## 总模型

当前规范默认覆盖三类 worker 形态：

- delegated subagent
  - 由 leader 在当前任务链路内显式委派
  - 可以同步完成，也可以转成后台 task
- background agent
  - 仍然属于 delegated agent，只是结果经 task / notification 回流
- teammate
  - 长生命周期 worker
  - 有稳定 team identity
  - 与 leader / 其他 teammate 通过 mailbox 或 viewed-input 通道交互

## 关键术语

- `Leader`
  发起委派、管理 worker、接收结果或通知的一方
- `WorkerAgent`
  被 leader 管理的执行单元总称
- `Subagent`
  临时 delegated worker
- `Teammate`
  team-bound 长生命周期 worker
- `TeammateExecutor`
  teammate 的 backend 抽象
- `Mailbox`
  teammate 间 durable point-to-point 消息通道
- `InterAgentMessage`
  任意一条 agent 间通信的高层抽象
- `ViewedTranscript`
  leader 正在查看的某个 worker 的 transcript projection

## 与其它子域的边界

- `task/`
  解决执行对象如何注册、跟踪、恢复和回收
- `multi-agent/`
  解决多个 worker 如何被启动、隔离、通信、投影给 leader
- `gateway/`
  解决外部 channel 与 harness worker 的边界
- `orchestration/`
  只负责 cloud control plane，不定义 local multi-agent 本体

## 目录内文档

- [agent-delegation.md](agent-delegation.md)
- [teammate-execution.md](teammate-execution.md)
- [message-routing.md](message-routing.md)
- [view-and-transcript-projection.md](view-and-transcript-projection.md)
