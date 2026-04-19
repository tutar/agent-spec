# Task Lifecycle

## 职责

本文统一 `Harness` 域内 task 的生命周期语义。

它回答的问题是：

- task 何时创建、注册、进入运行态和进入终态
- task 输出如何被稳定暴露与增量消费
- progress 与 terminal notification 如何分离
- foreground/background 与 UI 持有语义如何建模
- task 何时可以恢复、何时可以回收

本文讨论的是 task lifecycle 语义，不替代 [task-manager.md](task-manager.md) 中的 manager 职责与最小接口，也不替代 [../runtime-core/agent-runtime.md](../runtime-core/agent-runtime.md) 中的 turn 级 `terminal state`。

## 解决的问题

- 如何让 shell、agent、workflow、monitor、verifier 等不同执行对象共享同一套 task lifecycle
- 如何让宿主在不理解具体执行器内部细节的前提下，统一观察状态、读取输出和终止任务
- 如何让 restore、detail view、background session、notification queue 依赖统一 task 语义
- 如何避免“大输出只能靠内存持有”或“终态通知重复发出”这类实现分叉

共享对象模型见 [task-model.md](task-model.md)。

## 生命周期

### 标准状态

推荐最小状态机：

```text
pending -> running -> completed
pending -> running -> failed
pending -> running -> killed
```

语义：

- `pending`
  已注册但尚未稳定开始执行
- `running`
  正在推进，允许追加 progress 与输出
- `completed`
  正常结束，且已产生稳定结果
- `failed`
  因错误结束，未达到语义目标
- `killed`
  因显式终止而结束

这些状态只描述 task 自己的执行状态，不构成 `AgentRuntime terminal state` 的子集或别名。

### 注册

- task 应先注册到 `TaskRegistry`，再进入可观察生命周期
- `TaskRegistry` 是 task state 的单一事实来源
- 外部观察者应通过 registry 查询 task，而不是持有进程内临时对象引用

### 更新

- task 的状态迁移必须通过统一 lifecycle 写入口发生
- direct-call 便利接口如 `complete()`、`fail()`、`kill()`，也必须能严格推导为同一条 lifecycle 事件流

### 终态收敛

- 一个 task 最终只能收敛到一个 terminal status
- terminal status 一旦稳定，不得再回退到 `running`
- terminal cause 不得只存在于瞬时日志中，必须能通过 task state、terminal event 或 output handle 恢复

如果宿主需要更细粒度 stop reason，应作为 task event 或 terminal metadata 暴露，而不是改写基础 `TaskStatus` 集合。

## 输出与增量消费

task 输出必须通过稳定的 output handle 暴露，而不是假设调用方总能直接访问进程内缓冲。

必须满足：

- 调用方可以通过 `output_ref` 定位输出
- 调用方可以通过 `cursor` 或 offset 增量消费输出
- 大输出不应要求全量内存驻留
- output store 不得吞掉 terminal cause、关键错误或最后一次稳定结果

推荐语义：

- `file`
  默认本地实现；适合 tail、增量读取与恢复
- `stream`
  适合短生命周期或 transport-native streaming，但仍需提供可恢复 cursor
- `remote_log`
  适合远端 worker / cloud session 的输出代理
- `transcript_branch`
  适合把输出作为对话或执行分支的一部分 durable 保存

约束：

- output transport 可以变化，但 `output_ref + cursor` 语义必须稳定
- task 不应要求上层每次都重新全量读取输出
- output store 的存在是为了观察与恢复，不应替代 terminal state 本身

## 通知与恢复

### Progress 与 terminal notification 分离

- progress notification 应表示运行中事实，不应隐式宣称 task 已终态
- terminal notification 必须独立于 progress notification
- completed/failed/killed 的最终通知必须可去重

### 通知去重

`notified` 应作为 terminal notification 的闸门：

- 未进入终态前，`notified` 不应被当作完成标志
- 进入终态后，只允许一次稳定 terminal notification 成功提交
- kill path、finally path、restore replay 不得导致双发

### 恢复语义

restore 后，调用方至少应能重建：

- 最近一次 task status
- 已知 output handle 与 cursor
- 最近一次 terminal-like stop reason
- 当前是否仍可观察、可终止或仅可查看结果

如果宿主支持 background session 或跨进程接管：

- task identity 必须稳定
- output handle 必须在 restore 后继续可读，或能被显式判定为不可恢复

## Foreground、Background 与持有

foreground/background 是 task 的运行模式投影，不是另一种 task class。

推荐语义：

- foreground
  执行仍附着在当前互动上下文，但已经具备可 task 化的统一观察语义
- background
  执行脱离当前前台交互继续运行，仍由同一 task lifecycle 托管

task 可以被外部观察者持有，例如：

- detail view
- transcript view
- restore session
- notification queue consumer

约束：

- 被观察者持有的 terminal task 不得立即回收
- 持有释放后，宿主可按 `evict_after` 或等价策略延迟清理
- 前后台切换不应改变 task identity，也不应重建另一套状态机

## 回收

task 回收应晚于 task 完成。

推荐最小条件：

- task 已处于 terminal status
- terminal notification 已稳定完成
- 没有活跃观察者持有该 task
- grace period 或 `evict_after` 已满足

约束：

- 回收是资源管理行为，不得改变已经投影给外部的 terminal 事实
- output 清理与 task state 驱逐可以分阶段完成，但都不得早于恢复语义边界

## 与其它规范的边界

- `TaskManager` 的最小对象与接口见 [task-manager.md](task-manager.md)
- task 的共享对象模型见 [task-model.md](task-model.md)
- background agent 如何映射到 task lifecycle，见 [background-agent.md](background-agent.md)
- `AgentRuntime` 的 turn 级 `terminal state` 见 [../runtime-core/failure-and-terminal-states.md](../runtime-core/failure-and-terminal-states.md)
- 团队协作型任务列表不在本文范围内，见 [work-allocation.md](work-allocation.md)

## 规范结论

- task 必须是 harness 一等执行对象，而不是实现内部细节
- task 必须支持稳定 identity、统一状态机、输出句柄、通知去重、终止与恢复
- `output_ref + cursor` 应被视为稳定接口，具体输出后端只是实现选择
- terminal notification 必须与 progress 分离，并具备严格去重语义
- foreground/background 是 task 的运行模式，不是新的 task 类型
- task 完成不等于立即可回收；恢复边界与观察者持有必须优先于资源回收
