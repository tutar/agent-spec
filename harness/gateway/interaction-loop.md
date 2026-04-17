# Interaction Loop

## 职责

`InteractionLoop` 规范 gateway 与 harness 之间的标准交互循环。

它不是 CLI 专属的 `REPL` 术语，但可以把 CLI REPL 视为它的一种具体实现。

这条循环可概括为：

1. `Input`
   gateway 接收输入
2. `Eval`
   harness 推进 turn
3. `Projection`
   gateway 投影 runtime 事件
4. `Continuation`
   绑定 session，等待下一轮输入

## 稳定接口

推荐最小接口：

```text
GatewayInteractionLoop
  - accept_input(inbound_envelope) -> interaction_ref
  - bind_session(interaction_ref) -> session_ref
  - invoke_turn(session_ref, inbound_envelope) -> runtime_events
  - project_runtime_events(runtime_events) -> egress_events
  - await_next_input(session_ref)
```

也可拆为四个角色：

```text
Gateway
HarnessTurnRunner
EgressProjector
ContinuationController
```

## 设计要求

- 该循环必须是 channel-agnostic 的
- CLI REPL 只是默认实现，不应成为规范术语绑定
- `Eval` 必须由 harness 承担，而不是 gateway
- `Projection` 必须由 gateway 协调，但不要求所有 egress 都同步完成
- continuation 应绑定到 session，而不是某个 UI 组件
- supplement input 必须归入当前 interaction，而不是新建 chat 或新 session
- interrupt 必须保持显式 control 语义，与 supplement input 分离

## 默认实现映射

当前仓库中的 CLI 默认实现可映射为：

- `Input`
- `Eval`
- `Projection`
- `Continuation`

remote-control / bridge 变体则映射为：

- `Input`
- `Projection`
- `Transport`

## 规范结论

- 规范层应写 `InteractionLoop`，而不是把 `REPL` 当成通用概念
- CLI REPL 可以作为该循环的默认实现示例
- 任何 gateway 接入最终都应落入同一条 interaction loop 语义
