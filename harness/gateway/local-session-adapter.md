# Local Session Adapter

## 职责

`LocalSessionAdapter` 负责把本地 runtime 会话承载为 gateway 可管理的 session handle。

它解决：

- 如何启动本地 session
- 如何读取本地输出
- 如何转发 stdin/control
- 如何把本地执行封装成统一会话句柄
- 如何让 gateway 侧承载句柄与 `HarnessInstance` / `AgentRuntime` 分层

## 稳定接口

```text
LocalSessionAdapter
  - spawn(session_opts) -> session_handle
  - write_input(session_handle, input)
  - kill(session_handle)
  - observe(session_handle) -> runtime_events

SessionHandle
  - session_id
  - done
  - activities
  - current_activity
  - harness_instance_id?
  - write_stdin()
  - update_access_token()
```

## 设计要求

- 不要求本地 runtime 与当前进程内 harness 同构
- 可以是 in-process，也可以是 child process
- session handle 必须统一暴露活动、终止与输入写入能力
- 它位于 gateway 一侧，不应与 harness 本体混写
- `SessionHandle` 是 gateway-side carrier，不等于 `HarnessInstance`
- `HarnessInstance` 是 worker identity，不等于 `AgentRuntime`

## 默认实现映射


## 规范结论

- gateway 不应假设本地 runtime 只能以内嵌函数调用方式存在
- 本地 session handle、harness instance 和 internal runtime loop 必须作为三层不同概念保留
