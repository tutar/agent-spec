# Case: Verification Task Lifecycle And Verdict Attachment

## 目标

验证 correctness verification 被建模成独立 verifier task，并且其结构化 verdict 可稳定 attach 回主链路。

当前语义锚点：

- `harness/task/verification.md`
- `harness/task/task-model.md`
- `harness/task/task-lifecycle.md`

## Preconditions

- runtime 支持把 verification 建模成独立 verifier task 或语义等价执行单元
- task manager 或等价组件支持按 verifier handle 追踪状态与输出
- verification result 可 attach 回 session、task 或语义等价主链路对象

## Ingress

1. 提交一个需要独立复核的非平凡改动或执行目标
2. 触发 verification，自动或显式 spawn verifier
3. 观察 verifier 从创建到运行再到终态的状态变化
4. 读取 verifier 输出与结构化 verdict
5. 将 verification result attach 回原 session 或原 task
6. 模拟 restore / resume，验证 verdict 与 evidence 仍可追溯

## Expected Lifecycle

- verifier 拥有独立 task handle、status 与 output reference
- verifier 可进入 `running`，并最终收敛到单一 terminal status
- verifier 可以被等待、取消，或在 restore 后重新附着观察

## Expected Runtime Semantics

- verification 不得退化成主 agent 的自评完成
- verdict 至少可区分 `PASS / FAIL / PARTIAL`
- verification result 至少包含 verdict、evidence 与 findings 或语义等价结构
- evidence 应来自真实执行，而不是仅靠代码阅读或口头判断
- command-like verification trigger 与 verifier task lifecycle 必须分层

## Expected Persistent Effects

- verifier 输出可通过稳定 output handle 或等价 durable reference 追溯
- attach 后的 verification result 不因主链路继续推进而丢失
- restore 后仍可重新读取 verifier terminal status、输出句柄与最终 verdict

## Allowed Variance

- verification 可以自动触发，也可以显式触发
- verifier 可以本地执行，也可以由远端 worker 执行
- attach 位置可以是 session、task、review record 或语义等价对象

## Failure Conditions

- verification 只表现为主 agent 自我声明“已验证”
- verifier 没有独立 task identity 或 lifecycle
- verdict 不是结构化结果，无法区分 `PASS / FAIL / PARTIAL`
- attach 后无法再追溯 evidence、findings 或最终 verdict
- restore 后 verifier 结果漂移、丢失，或必须重新执行才能读取
