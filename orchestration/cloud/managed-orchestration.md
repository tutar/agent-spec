# Managed Orchestration

本文档位于 `cloud/` 目录下，因为它描述的是 Cloud deployment 下的核心控制面风格，而不是所有 hosting profile 的共同主路径。

## 目标

`Managed Orchestration` 描述托管 agent 系统如何在 session、harness 和 hands 解耦后仍保持可靠、低延迟和可扩展。

它关注的是系统级编排，而不是单次 tool 调用细节。

## 要解决的问题

- 如何同时管理多个 brains 和多个 hands
- 如何把本地、后台、远端、子代理任务统一进一套生命周期
- 如何在同一系统里同时容纳 sync worker、background task、persistent teammate、remote task
- 如何让 hand 冷启动不拖慢纯推理任务
- 如何让 harness 与 hands 各自失效后仍可恢复

## 核心能力

- many brains
  多个 harness 实例可横向扩展
- many hands
  一个 brain 可同时调度多个 execution targets
- lazy provisioning
  不在 session 启动时预创建全部 hands
- wake and resume
  harness 崩溃后可基于 session log 继续
- remote resources
  可接入 VPC、远端系统和非本地 execution target

## Many Brains

brain 应被设计为可替换、可重启、可无状态扩容的 harness 实例。

要求：

- harness 不应把唯一状态只保存在内存
- 新 harness 应能通过 session log 继续工作
- 同一 session 不应依赖某一个“宠物”进程

## Many Hands

hands 应是可枚举、可路由的 execution targets。

要求：

- 一个 brain 可以拥有多个 hands
- hands 可以来自不同环境、网络和信任域
- hands 可以按需创建，不需要 session 启动即全量存在

## Lazy Provisioning

规范应支持：

- session 先启动模型推理
- 只有在需要某类 hand 时才触发 `provision`
- 不需要 hand 的任务不为 hand 的冷启动付出 TTFT 成本

这要求 orchestration 层能把 inference 启动与 execution provisioning 解耦。

## Failure Recovery

需要明确三类恢复：

- harness failure
  新 harness 通过 session log 恢复
- hand failure
  作为 tool-call 或 execution failure 回写，并可重 provision
- network/control failure
  不应导致 session durable state 丢失

## Remote And VPC Scenarios

托管编排不应假设：

- harness 与资源同网段
- harness 与 hands 共容器
- 用户必须把全部资源迁移到平台内部

因此应允许：

- 远端 sandbox
- VPC 内 hands
- customer-managed execution targets
- proxy-based external tools

## 性能目标

规范应支持以下优化空间：

- 低 TTFT
- inference 与 hand provisioning 解耦
- cold start 不阻塞纯推理 session
- 多 hand 环境下的选择与复用

## 与现有模块的边界

- `AgentRuntime`
  处理单个 harness 内部 loop
- `ToolExecutor`
  处理 tool invocation orchestration
- `Sandbox / Hands`
  提供执行能力
- `Managed Orchestration`
  处理这些对象在托管系统中的连接、扩展、恢复与分配

## 默认实现

当前代码库中的默认 orchestration 实现是“task-first orchestration”：

  负责汇总当前支持的 task types
  承担本地子 agent 生命周期
  承担远端 agent 生命周期
  作为默认的 subagent orchestration entry

默认模式的特点是：

- orchestration 主要围绕 task registry 展开
- local shell、local agent、remote agent 是三类主要默认执行对象
- 已经具备多种 agent orchestration modes 的雏形，只是尚未在规范中显式命名
- already has the shape of many-brains/many-hands orchestration, but still primarily optimized for a single product runtime

## 规范结论

- managed-agent 规范必须显式支持 many brains / many hands
- orchestration 层必须允许 harness 和 hands 独立演进
- lazy provisioning 和 wake-based recovery 应被视为一等能力
