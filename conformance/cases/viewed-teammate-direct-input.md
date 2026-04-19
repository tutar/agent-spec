# Case: Viewed Teammate Direct Input

## 目标

验证 leader 在 viewed transcript 中输入内容时，消息会直接进入目标 worker，而不是主线程 prompt 流。

## Preconditions

- runtime 支持 viewed transcript
- viewed worker 可接受 direct input 或语义等价的 targeted input

## Ingress

1. leader 打开某个 teammate 或 worker 的 viewed transcript
2. leader 发送一条输入
3. 观察消息被哪一方消费

## Expected Runtime Semantics

- 输入被投递给当前 viewed worker
- 该输入不会进入 leader 的普通主线程 prompt 流
- 非当前 viewed worker 不会消费该输入

## Failure Conditions

- viewed 输入回到了 leader 主线程
- viewed 输入被错误投递给其它 worker
- runtime 无法区分 viewed direct input 与普通 prompt
