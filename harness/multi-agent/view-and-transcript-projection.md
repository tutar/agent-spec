# View And Transcript Projection

## 职责

本文定义 leader UI 如何查看多个 worker，而不把 viewed transcript 与 worker 执行本体混为一层。

## 核心概念

- `ViewedTranscript`
  leader 当前正在查看的某个 worker transcript projection
- `viewing_agent_task_id`
  当前被查看的 worker 标识
- `retained task`
  因 viewed transcript 仍被持有而暂不回收的 task

## 分层原则

- viewed transcript 是 projection，不是 worker 本体
- task state 是执行层对象
- mailbox 是 inter-agent messaging
- durable transcript / sidechain 是 session 层引用

因此：

- `messages` 可以只是 UI mirror
- 完整 durable history 可以来自 transcript、sidechain、output store 或 mailbox
- viewed transcript 不应被当作 worker 的唯一真实记录

## 保留与回收

leader 正在查看某个 worker 时：

- 对应 task 不应被立即驱逐
- 可以进入 retained 状态
- 退出 viewed transcript 后，才允许进入正常 eviction 路径

## 直接输入

viewed transcript 下的输入是 targeting 当前 worker 的 direct injection path。

约束：

- 这条输入不应回到 leader 的普通 prompt 流
- 这条输入不应被误投递到其它 worker
- viewed direct input 与 mailbox 应保持独立语义

## 规范结论

- viewed transcript 只是多 agent 系统的 UI projection
- retained / eviction 行为必须尊重 leader 是否仍在查看该 worker
- viewed direct input 是单独的消息通道，不等于 mailbox 或主线程 prompt
