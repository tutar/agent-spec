# Case: Inter-Agent Task Notification Routing

## 目标

验证 task notification 在多 agent 场景下按目标 worker 定向投递，而不是广播。

当前语义锚点：

- `harness/multi-agent/message-routing.md`
- `harness/multi-agent/README.md`

## Preconditions

- runtime 支持多个并发 worker
- task notification 通过共享队列或语义等价机制回流
- 每个 worker 都有稳定 identity

## Ingress

1. 启动 leader 和至少两个 worker
2. 向其中一个 worker 发送 task notification
3. 观察各 worker 的 notification 消费结果

## Expected Runtime Semantics

- 目标 worker 可以消费到该 notification
- 非目标 worker 不会消费到该 notification
- leader thread 与 worker 的 notification scope 明确区分
- task notification 与 mailbox、direct view input 不共享同一 delivery 语义

## Failure Conditions

- notification 被所有 worker 同时看到
- 非目标 worker 消费了不属于自己的 notification
- 无法根据 worker identity 限定 notification recipient
