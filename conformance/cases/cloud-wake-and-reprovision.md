# Case: Cloud Wake And Reprovision

## 目标

验证云端宿主下的 `wake / reprovision / resume` 语义。

当前语义锚点：

- `orchestration/cloud/README.md`
- `orchestration/cloud/managed-orchestration.md`
- `harness/deployment-boundaries.md`

## Preconditions

- host profile 为 `Cloud`
- session durable state 与 execution environment 解耦
- runtime 支持 wake/resume
- runtime 支持 execution target reprovision 或 hand reprovision

## Ingress

1. 创建一个需要远端 execution 的 session
2. 在 session 挂起后发送新的 wake ingress
3. 模拟 execution environment 失效
4. 触发 reprovision，并继续后续 turn

## Expected Lifecycle

- session 可以从 dormant/idle 状态被 wake
- execution environment 失效后，session 不应被判定为丢失
- reprovision 后 turn 可继续，或以结构化 failure 结束

## Expected Runtime Semantics

- `wake` 作用于 session binding，不等同于新建 session
- `reprovision` 作用于 execution environment，不应改变 session identity
- durable session history 与 environment lifecycle 必须解耦

## Expected Persistent Effects

- session id 保持稳定
- event log 记录 wake、reprovision、continuation 或 failure
- 失败时仍应有可恢复或可审计记录

## Failure Conditions

- reprovision 导致 session identity 改变
- environment 丢失被错误等同为 transcript 丢失
- wake 后创建了新的逻辑会话而不是恢复旧会话
