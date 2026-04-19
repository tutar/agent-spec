# Message Routing

## 职责

本文定义 local multi-agent 系统中的消息路由通道。

## 路由通道

当前规范默认存在三类通道：

- `task_notification`
  - leader / subagent 之间的 task 结果与通知回流
- `mailbox`
  - teammate 与 leader 或其它 teammate 的 durable point-to-point 消息
- `direct_view_input`
  - leader 在 viewed transcript 中直接向目标 worker 注入输入

## 推荐最小对象

```text
InterAgentMessage
  - channel: task_notification | mailbox | direct_view_input
  - sender
  - recipient
  - team?
  - payload
  - summary?
  - timestamp
```

## 1. Task Notification Routing

task notification 的特点是：

- 队列是进程级共享结构
- 但消费必须按 recipient scope 过滤
- main thread 只消费发给主线程的项
- subagent 只消费发给自己的 task-notification

约束：

- task notification 不是广播通道
- 子 worker 不得读到不属于自己的 notification

## 2. Mailbox Routing

mailbox 的特点是：

- durable
- point-to-point
- agent-scoped inbox

推荐最小消息字段：

```text
MailboxMessage
  - from
  - to
  - team
  - text
  - timestamp
  - read
  - summary?
  - color?
```

约束：

- mailbox 默认不是共享群聊 transcript
- read / unread 状态必须可追踪
- 发送方和接收方必须能从 identity 唯一定位

## 3. Direct View Input

当 leader 正在查看某个 worker 的 transcript 时，用户输入可以直接注入目标 worker。

这条通道的特点是：

- 是 UI-bound input path
- 目标是当前 viewed worker
- 不经过普通主线程 prompt 流
- 不等于 mailbox

## 路由原则

- 每个通道都必须有明确 recipient scope
- mailbox、task notification、direct view input 不得混成同一 delivery 语义
- leader 不应自动看到 teammate 全量输出，除非收到显式消息或进入 viewed transcript

## 规范结论

- local multi-agent 至少应区分 task_notification、mailbox、direct_view_input 三类通道
- 队列共享不等于消息广播；recipient scoping 必须显式存在
- teammate 默认通过显式消息而不是隐式 stdout 回流与 leader 协作
