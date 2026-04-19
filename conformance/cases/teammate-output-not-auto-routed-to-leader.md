# Case: Teammate Output Not Auto Routed To Leader

## 目标

验证 teammate 的普通执行输出默认不会自动回灌给 leader。

## Preconditions

- runtime 支持 leader + teammate 模型
- teammate 可以独立运行一个可观察任务

## Ingress

1. 启动一个 teammate
2. 让 teammate 产生普通执行输出
3. leader 不发送显式查看请求，也不接收显式 teammate message

## Expected Runtime Semantics

- teammate 可以继续推进自己的工作
- leader 不会自动收到该 teammate 的全量输出
- 若需要回流，必须通过显式 message、notification 或 viewed transcript 通道

## Failure Conditions

- teammate 的普通输出被隐式注入 leader transcript
- leader 无显式动作却自动看到 teammate 全量会话内容
