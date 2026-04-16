# Case: Memory Consolidation Background Safety

## 目标

验证 background / delayed memory consolidation 在失败、锁冲突或延迟时，不会破坏 session restore 与 durable memory 视图。

## Preconditions

- session 支持 memory extraction
- memory subsystem 支持 consolidation 或 dream-style background consolidation
- consolidation 支持 lock 或等价并发控制

## Ingress

1. 完成一轮会产生 durable memory 候选的交互
2. 启动一次 background consolidation
3. 在 consolidation 过程中模拟失败、锁冲突或中断
4. 对原 session 执行 resume，并再发起一轮 recall

## Expected Runtime Semantics

- session restore 正常
- 未 commit 的 consolidation 结果不会提前进入 recall
- 已 commit 的 durable memory 仍然可被正常 recall
- 下次 consolidation 允许安全重试

## Failure Conditions

- consolidation 失败导致 session checkpoint / resume 失效
- 未 commit 的 memory 被错误 recall
- lock 冲突导致 durable memory 进入部分提交状态
