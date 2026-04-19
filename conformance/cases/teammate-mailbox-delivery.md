# Case: Teammate Mailbox Delivery

## 目标

验证 teammate 之间的 mailbox 是 durable、point-to-point 的消息通道。

## Preconditions

- runtime 支持 teammate identity
- teammate 之间存在 mailbox 或语义等价 inbox
- mailbox 支持 read / unread 或等价已消费状态

## Ingress

1. 启动 leader 与至少两个 teammate
2. 由 leader 向其中一个 teammate 发送 mailbox message
3. 再由该 teammate 向另一个 teammate 发送 mailbox message

## Expected Runtime Semantics

- 每条消息只投递给目标 recipient
- recipient 可读取未读消息
- 已消费状态可追踪
- mailbox 不要求 leader 自动看到消息正文，除非 leader 是 recipient 或显式查看

## Failure Conditions

- mailbox 退化为共享广播
- 无法区分已读与未读
- 无法通过 agent identity 唯一定位 recipient
