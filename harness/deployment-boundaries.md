# Deployment Boundaries

## 目标

这一页定义 agent runtime 在不同部署拓扑下的稳定约束。

它回答的不是 UI 形态差异，而是更底层的问题：

- 组件是否共进程
- 组件是否共主机
- 组件是否可以独立部署、独立升级、独立恢复
- 上层 agent 集成的是默认实现，还是稳定接口

如果这些问题不先写清，`Local`、`Cloud` 虽然都能复用同一套术语，但程序语义会逐步分叉。

## 核心结论

agent runtime 必须先按 `out-of-process / remote-capable` 语义设计，再提供 `in-process` 优化实现。

换句话说：

- `in-process` 是一种 binding 优化，不应成为接口默认语义
- `Cloud` 不是 “Local runtime + 远程工具”
- `Local` 不等于单进程；它只是单机部署，仍然允许多进程或本地 IPC
- 只有当接口在跨进程场景下仍然成立时，两种 hosting profile 才真正共享同一套 agent 语义

## 两层模型

推荐把系统拆成两层：

### 1. 语义层

语义层定义稳定的领域契约，只表达能力与状态机，不表达调用方式。

例如：

```text
Harness.run_turn(input, session_handle) -> event_stream, terminal_state
Session.append_events(session_handle, events) -> checkpoint
Session.load_slice(session_handle, cursor) -> session_slice
ToolExecutor.execute(tool_call, execution_context) -> tool_event_stream
TaskManager.spawn(task_spec) -> task_handle
TaskManager.await(task_handle) -> task_terminal_state
Sandbox.invoke(action, sandbox_handle) -> execution_result
```

这些接口的语义必须天然支持：

- 异步
- 流式事件
- 部分失败
- 超时
- 取消
- 重试
- 幂等键
- handle / checkpoint / cursor / job id

### 2. Binding 层

binding 层负责把语义层投影到具体部署方式：

- `InProcessBinding`
  适用于 Local 场景下的同进程优化
- `LocalIpcBinding`
  适用于 Local 场景下的多进程本机部署
- `RemoteServiceBinding`
  适用于 Cloud control plane、worker、remote sandbox

binding 可以改变性能、延迟和部署方式，但不应改变领域语义。

## Core Contract 与 Convenience Contract

不建议把 Local 和 Cloud 设计成两套平行接口。

更稳妥的方式是：

- 只有一套 `core semantic contract`
- 在其之上提供不同宿主可用的 `convenience API`

推荐关系如下：

```text
Core streaming contract
  -> 表达真实运行时语义

Convenience direct-call contract
  -> 只是对 core contract 的包装与聚合
```

例如：

```text
Core:
  Harness.run_turn_stream(...) -> AsyncIterable<RuntimeEvent>

Convenience:
  Harness.run_turn(...) -> Promise<TerminalState>
```

这里的 `run_turn()` 不应拥有额外语义。
它只能做这些事情：

- 消费完整个 `run_turn_stream()`
- 聚合 terminal state
- 对常见事件做本地缓存或格式化

它不应做这些事情：

- 引入只有本地 direct-call 才有的额外状态跳转
- 隐藏 cloud 才会出现的部分失败/取消/超时语义
- 让同步调用和流式调用产生不同的完成条件
- 引入与 event stream 不一致的错误分类

## 为什么不应拆成两套正式接口

如果把它写成：

- Cloud: event-stream contract
- Local: direct-call contract

那么即使字段相同，后面也会逐步分叉：

- completion 定义不同
- error surface 不同
- cancellation 语义不同
- progress 语义不同
- recovery / retry 语义不同

最终会演化成两套 agent，而不是一套可移植的 agent。

因此更合适的规则是：

- streaming contract 是第一性接口
- direct-call contract 是 convenience facade
- 任何 direct-call API 都必须可由 streaming API 严格推导
- 如果某个信息无法从 event stream 得到，它就不应只出现在 convenience API 里

## 为什么必须这样分

如果不做这层拆分，程序里会出现看起来“可替换”、实际却不兼容的接口：

- 本地实现把 `Session` 当作进程内可变对象，cloud 却需要 durable append-only store
- 本地实现把 `ToolExecutor.execute()` 当作同步函数，cloud 却需要 job / stream / retry
- 本地实现把 `TaskManager` 当作内存里的 task registry，cloud 却需要可恢复的 task handle
- 上层 agent 直接 import 默认实现，结果和未来的远端实现耦合到一起

这会带来四类系统性问题：

- 恢复语义不一致
- 错误处理模型不一致
- 背压 / streaming 行为不一致
- 版本升级和组件替换成本过高

## 接口设计硬约束

凡是未来可能跨进程、跨主机、跨部署边界的组件接口，都不应泄漏进程内语义。

因此不应把这些东西作为稳定契约：

- 共享内存对象引用
- 同步阻塞调用作为唯一执行模型
- 隐式全局状态
- 本地事务一致性假设
- “函数返回即提交成功”的假设
- 只有进程内才成立的回调时序

更适合进入稳定契约的是：

- append-only event
- command + event
- checkpoint / cursor
- durable handle
- lease / owner / heartbeat
- timeout / cancellation / retry contract
- capability negotiation
- explicit terminal state

## 对各组件的约束

### Harness

- 不应假设 `Session`、`ToolExecutor`、`Sandbox` 与自己共进程
- 应优先依赖 `session_handle`、`checkpoint`、`event_stream`
- 崩溃后应允许由新的 harness 实例接管
- 如果同时提供 direct-call API，它也应只是 `event_stream` API 的包装

### Session

- 应优先暴露 append/read/resume 能力，而不是暴露可变内存结构
- 应把 durable log、checkpoint、replay 作为一等能力
- 应允许 runtime 不共生

### ToolExecutor

- 不应假设工具调用总能同步完成
- 应允许 `started/progress/result/failed/cancelled` 事件流
- 应允许工具结果先落 durable state，再投影回 turn
- 如果本地实现暴露同步或 `await` 风格封装，它不应弱化这些事件语义

### TaskManager / Orchestration

- task 的稳定标识应是 `task_handle`，不是进程内对象引用
- `TaskManager` 负责 harness 内的 detached/background task
- `Orchestration` 负责 cloud control plane 下的 remote wake / reprovision / reattach
- 二者都应把 resume、reattach、recover 视为标准路径，而不是异常路径

### Sandbox / Hands

- 应把 sandbox 看成可替换执行面，而不是 harness 内部函数库
- capability、lease、network boundary、credential boundary 应显式声明
- Cloud profile 下应默认允许 remote provisioning 和 reprovision

## 可替换执行面

`Harness` 与 `Sandbox` 都应被视为可替换执行面，而不是必须长期存活的 stateful singleton。

这意味着：

- `Cloud` 下重启、扩缩容或重新创建 harness/sandbox 实例，不应破坏已有 session 或 task 的恢复能力
- `Local` 下应用重启或系统重启后，也不应破坏已 durable session 的 resume 能力
- resume 依赖的是 `Session`、checkpoint、task handle、event log 等 durable 边界，而不是某个仍然存活的进程实例
- 若某个组件被替换，新实例应能通过 stable handle / checkpoint / restore store 继续接管工作

## 对上层 agent agent 集成者的含义

上层 agent 不应默认“直接依赖一组库内单例实现”。

更安全的集成方式是：

- 上层 agent 依赖稳定接口
- 通过 profile/binding 注入默认实现
- 在 Local 下可以选择 in-process binding 或 local IPC binding
- 在 Cloud 下可以选择 remote binding
- 如果某个组件需要替换为自定义实现，应替换该组件的 binding 或 adapter，而不是修改 harness 核心语义
- 如果希望暴露更简单的本地 API，应把它实现成 convenience facade，而不是再定义一套平行语义

## Hosting Profile 解释

两种 hosting profile 的真正区别，应理解为部署拓扑差异，而不只是交互壳差异。

- `Local`
  偏向单机部署，可使用同进程或本地 IPC 优化，但不应因此把接口设计成只能单机使用
- `Cloud`
  应被视为默认的“最坏情况”约束来源：分布式、可恢复、远程执行、独立故障域

因此，profile 的关系不应是：

```text
Shared semantic layer
  + InProcessBinding => Local
  + LocalIpcBinding => Local
  + RemoteServiceBinding => cloud
```

## 规范结论

- `hosting profile` 首先是 deployment topology，不只是 UI 形态
- 组件接口必须先满足 distributed semantics，再允许本地优化
- 默认实现可以是单进程，但默认语义不能是单进程
- 上层 agent 应依赖稳定接口与 binding，而不是绑定某个默认实现的对象模型
- 如果某个接口无法在 remote/distributed 场景下成立，它就不应成为 agent 的核心稳定接口
- direct-call API 可以存在，但必须是 core streaming contract 的严格包装
