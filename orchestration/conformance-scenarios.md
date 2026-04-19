# Conformance Scenarios

## 目标

本文件定义 `Orchestration` 模块下的 managed-agent 架构关键合规场景。

## 必测场景

### 1. Harness Crash Recovery

- 触发条件：harness 在 turn 中途崩溃
- 验收：新 harness 可通过 session log 恢复并继续

### 2. Sandbox Reprovision

- 触发条件：execution target 失效
- 验收：task/session 不丢失，系统可重 provision 并继续或失败回写

### 3. Many Brains

- 触发条件：多个 harness 实例处理同类请求
- 验收：session durable state 不依赖单一宠物进程

### 4. Many Hands

- 触发条件：一个 brain 面向多个 execution targets
- 验收：hand routing 正确，失败隔离正确

### 5. Lazy Provisioning

- 触发条件：无 execution 需求的 turn
- 验收：不会为 hand 冷启动付出额外 TTFT 成本

## 规范结论

- managed-agent conformance 不应只测工具调用，还必须覆盖恢复、重建、many brains、many hands、lazy provisioning
