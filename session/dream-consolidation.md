# Dream Consolidation

## 职责

`Dream Consolidation` 是 durable memory 的周期性整合机制。

它的目标不是从单个 turn 提取记忆，而是对已有 memory files、近期 session 线索和历史索引做一次反思式整理。

它通常负责：

- 合并重复 memory
- 修正已过时或被证伪的 memory
- 将近期 session 中的稳定信号整合进 topic memory
- 修剪和重建 memory index

它不同于：

- `extract memories`
  偏 session-level 的增量提取
- `compact`
  偏上下文窗口治理
- `verification`
  偏任务正确性复核

## 触发模式

`Dream Consolidation` 至少应支持两种触发模式。

### 1. Manual Dream

由用户显式触发，例如：

- slash command
- skill invocation
- GUI action
- SDK API call

特点：

- 由用户明确要求开始
- 通常运行在主 interaction loop 或显式前台任务中
- 可使用标准用户权限语义

### 2. Auto Dream

由系统按策略自动触发。

典型 gate 包括：

- 时间门槛
- session 数量门槛
- memory activity 门槛
- consolidation lock

特点：

- 面向后台长期整合
- 通常作为异步 task 或后台 subagent 执行
- 应显式建模调度门槛与锁语义

## 稳定接口

推荐最小接口：

```text
DreamConsolidationRequest
  - trigger: manual | automatic
  - memory_scope
  - source_sessions[]
  - existing_memory_refs[]
  - index_ref?
  - consolidation_policy?

DreamConsolidationResult
  - updated_memory_refs[]
  - pruned_memory_refs[]
  - updated_index_ref?
  - summary
  - status: completed | no_change | failed | killed
  - trigger: manual | automatic

DreamScheduler
  - should_trigger(now, memory_state, session_activity) -> boolean
  - acquire_lock(scope) -> lock_ref?
  - release_lock(lock_ref)
  - record_completion(result)

DreamRunner
  - run_manual(request) -> result
  - run_automatic(request) -> task_or_result
```

## 规范要求

### 1. Manual 与 Automatic 必须共享同一 consolidation 语义

二者的差异应主要体现在：

- 触发来源
- 调度方式
- 权限与执行面

而不应体现在：

- 对 memory 的基本操作语义
- 对 index / topic memory 的处理目标
- 对最终结果对象的定义

推荐直接复用 [memory-consolidation.md](memory-consolidation.md) 中的 `ConsolidationResult` 语义，
仅额外补 `trigger` 或语义等价字段。

### 2. Dream 必须是 consolidation，不是普通 extraction

dream 的职责应聚焦于：

- cross-session 整理
- memory file merge
- stale fact correction
- index pruning

而不是简单把当前 turn 抽成一条新记忆。

### 3. Automatic Dream 必须显式建模调度门槛

自动 dream 不应在每轮 turn 无条件触发。

至少应允许：

- 最小时间间隔
- 最小 session 数量
- scan throttle
- in-progress lock

### 4. Automatic Dream 必须显式建模锁语义

若系统允许后台 consolidation，则必须处理：

- 并发 dream
- crash recovery
- aborted run rollback
- 下一次重试时机

Automatic Dream 的失败不得影响：

- session transcript restore
- durable memory 的已提交视图
- 后续普通 turn 的继续运行

### 5. Dream 应允许作为受限 subagent 执行

推荐权限面：

- unrestricted read of memory inputs and narrow transcript search
- edit/write only inside approved memory scope
- read-only shell exploration

不应默认允许它写业务代码或修改非 memory 区域。

## 与其它模块的边界

- 与 [memory-consolidation.md](memory-consolidation.md)
  `Memory Consolidation` 是上层能力；`Dream Consolidation` 是其中面向 cross-session memory 整理的标准模式
- 与 [memory-model.md](memory-model.md)
  `Dream Consolidation` 主要作用于 durable memory，而不是 short-term session memory
- 与 [../orchestration/agent-orchestration/task-manager.md](../orchestration/agent-orchestration/task-manager.md)
  `Auto Dream` 可作为后台 task 落地，但 task lifecycle 不等于 dream 语义本身
- 与 [../tools/command-surface/reflection-and-verification-commands.md](../tools/command-surface/reflection-and-verification-commands.md)
  `Dream` 是 memory reflection，不是 correctness verification

## 默认实现映射

当前仓库中的默认实现映射为：

- 自动调度与执行见 [../../services/autoDream/autoDream.ts](../../cc/services/autoDream/autoDream.ts)
- dream prompt 见 [../../services/autoDream/consolidationPrompt.ts](../../cc/services/autoDream/consolidationPrompt.ts)
- 锁与时间戳语义见 [../../services/autoDream/consolidationLock.ts](../../cc/services/autoDream/consolidationLock.ts)
- 后台任务表面见 [../../tasks/DreamTask/DreamTask.ts](../../cc/tasks/DreamTask/DreamTask.ts)
- memory 路径与 daily log/index 语义见 [../../memdir/paths.ts](../../cc/memdir/paths.ts)

当前代码库还明确保留了手动 `/dream` 概念：

- `recordConsolidation()` 的注释直接指出 manual `/dream`
- `autoDream.ts` 明确区分 manual `/dream` 与自动后台 dream 的工具约束与执行面

## 规范结论

- `dream` 应作为 session 域内的标准 consolidation 模式存在
- 规范应同时覆盖 `manual /dream` 与 `autoDream`
- `autoDream` 是调度模式，`dream` 是 consolidation 语义
- dream 的默认执行形态可以是受限 subagent + task surface
- `DreamConsolidationResult` 应与 `ConsolidationResult` 共享字段与终态语义
