# Case: Verification Command Vs Verifier Task Boundary

## 目标

验证 verification command-like capability 与 verifier task lifecycle 分层，而不是把 command trigger 与 verifier execution collapse 成同一层对象。

当前语义锚点：

- `tools/command-surface/reflection-and-verification-commands.md`
- `harness/task/verification.md`

## Preconditions

- runtime 支持 verification command-like capability、或语义等价的用户触发入口
- host 支持独立 verifier task、或语义等价的独立 verification execution unit
- verification result 可回流到 session、task 或语义等价主链路

## Ingress

1. 通过 command surface 触发一次 verification
2. 观察 command trigger 与 verifier execution 的关系
3. 等待 verifier 进入终态并读取结果
4. 验证 verification result 被 attach 回主链路

## Expected Runtime Semantics

- verification command 只是 trigger surface，不等于 verifier lifecycle 本身
- verifier 应拥有独立 task handle、status 和 output reference，或语义等价对象
- command completion 与 verifier terminal completion 可分离
- verification verdict 至少可区分 `PASS / FAIL / PARTIAL`
- verification command 不得绕过 verifier 的独立 evidence-based execution

## Expected Persistent Effects

- verifier result 至少以 stable reference、task output 或等价 durable surface 可追溯
- attach 后的 verdict 与 evidence 不会因 command trigger 结束而丢失

## Allowed Variance

- verification 可以是自动 spawn，也可以是显式 command 触发后再 spawn
- command 结果可以立即返回 accepted/started，也可以等待 verifier terminal 后再返回摘要

## Failure Conditions

- verification command 自身就是最终 verification 结果，没有独立 verifier execution
- verifier 没有独立 task-like identity
- command trigger 结束后无法追溯 verifier result
- verdict 不是结构化结果
