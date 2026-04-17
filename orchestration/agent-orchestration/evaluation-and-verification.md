# Evaluation And Verification

## 职责

`EvaluationAndVerification` 定义 orchestration 层如何执行 verification 这类 review capability，而不是如何把它暴露给 harness。

它负责：

- 将 verification 建模成独立 agent / task
- 为 verifier 分配 mode、工具边界与生命周期
- 等待、取消、恢复 verifier 执行
- 将验证结果 attach 回主编排链

它不负责：

- 定义 review / verification capability surface
- 把 verification 暴露成 command
- 通用 telemetry
- session lifecycle 投影
- tool-level tracing

## 稳定接口

推荐最小接口：

```text
VerificationRequest
  - target_session
  - target_agent?
  - original_task
  - changed_artifacts?
  - evidence_scope?
  - verification_policy?

VerifierTaskHandle
  - verifier_id
  - task_id?
  - mode
  - status
  - output_ref?

VerificationOrchestrator
  - spawn_verifier(request) -> verifier_handle
  - await_verifier(handle) -> verification_result
  - cancel_verifier(handle)
  - attach_verification(session_or_task, verification_result)

VerificationResult
  - verdict: PASS | FAIL | PARTIAL
  - evidence[]
  - findings[]
  - limitations[]
  - verifier_id
```

## 规范要求

### 1. 验证应由独立执行单元完成

不应将主 agent 自检视为充分验证。

### 2. verdict 必须结构化

至少支持：

- `PASS`
- `FAIL`
- `PARTIAL`

### 3. evidence 必须来自真实执行

代码阅读、推测和口头解释不能替代执行证据。

### 4. orchestration 应允许自动或显式触发 verifier

对非平凡改动，编排层应允许自动或半自动进入 verification 阶段。

### 5. verification command 与 verifier task 需要分层

`Tools` 可以把 verification 暴露成 command-like capability，但真正的子 agent / task 生命周期应由 orchestration 承担。

## 推荐默认策略

- 将 verifier 建模成独立 agent
- verifier 默认只读项目目录
- verifier 输出结构化 evidence + verdict
- 主 agent 在汇报完成前应等待 verifier 结果
- capability surface 由 [../../tools/command-surface/reflection-and-verification-commands.md](../../tools/command-surface/reflection-and-verification-commands.md) 定义

## 默认实现映射


## 规范结论

- agent 评估应优先通过独立 verifier 完成
- verifier task 应作为 orchestration 中的标准模式，而不是临时技巧
- verification command 与 verifier task 应明确分层
- 监控与评估必须分开建模
