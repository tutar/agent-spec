# Startup And Turn-Zero Context

## 职责

本页定义哪些上下文只在启动期注入，哪些需要按 turn 重算。

它解决的核心问题是：不要把 session-start、agent-start、turn-zero briefing、resume-start 混成同一种 prompt patch。

## 最小对象

```text
StartupContext
  - kind
  - first_use_only
  - reentry_policy
  - transcript_visibility
  - dedup_policy
  - payload
```

## 推荐分类

- `session_start`
  主会话刚开始时的启动上下文。
- `agent_start`
  delegated agent / teammate 首次启动时的本地 briefing。
- `turn_zero`
  在第一轮模型调用前提供的一次性 kickoff 信息。
- `resume_start`
  从 restore / reentry 恢复时重新宣布的上下文。

## 语义要求

### 1. session-start 与 agent-start 必须分开

`session_start` 面向主会话连续性。

`agent_start` 面向：

- worker role-local briefing
- delegation additions
- team-local coordination bootstrap

二者不能互相污染。

### 2. startup context 不是 bootstrap prompt

bootstrap prompt 是稳定 system skeleton。

startup context 是一次性或按 reentry 策略重放的运行时上下文。

### 3. turn-zero briefing 不是 agent prompt

首轮一次性说明不应通过修改 agent prompt 本体实现。

否则会导致：

- prompt cache churn
- 首轮/后续轮语义混淆
- agent identity 与 kickoff payload 混层

### 4. resume-start 必须显式建模

从 restore 或 UI reentry 回来时，某些上下文需要重新宣布：

- 当前执行模式
- pending action
- viewed transcript binding
- active worker identity

这些内容不能假设模型会从压缩后的历史中自动推断。

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

- startup payload 常来自本地 runtime state、当前 mode、delegation setup、UI 视图状态

### Cloud-Compatible Mapping

- startup payload 仍由 harness 组织
- 所需状态可来自远程 session/runtime state，但 lifecycle 语义不变

## 规范结论

- startup-only context 必须是独立语义层
- session-start、agent-start、turn-zero、resume-start 必须区分
- 一次性 briefing 不应污染 bootstrap prompt 或 agent prompt 本体
