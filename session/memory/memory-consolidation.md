# Memory Consolidation

## 职责

`Memory Consolidation` 负责把 session 中已产生的经验提炼为更稳定的长期记忆表示。

它不同于：

- `session transcript`
  原始会话事件
- `short-term session memory`
  面向 continuity 的短期摘要
- `compact`
  面向上下文窗口治理的重组

它的核心目标是：

- 从 session 或跨 session 经验中提炼 durable memory
- 把碎片化记忆整合成更可复用的长期结构
- 为 `manual /dream` 与 `autoDream` 提供统一的 consolidation 语义

## 稳定接口

推荐最小接口：

```text
MemoryExtractionRequest
  - session_ref
  - transcript_slice
  - extraction_policy?

MemoryConsolidationRequest
  - memory_scope
  - source_sessions[]
  - existing_memory_refs[]
  - consolidation_policy?

DreamConsolidationRequest
  - trigger: manual | automatic
  - memory_scope
  - source_sessions[]
  - existing_memory_refs[]
  - index_ref?
  - consolidation_policy?

MemoryConsolidator
  - extract(request) -> extracted_memories
  - consolidate(request) -> consolidated_memories
  - commit(memories) -> memory_refs
  - dream(request) -> dream_result
```

建议补充状态接口：

```text
ConsolidationState
  - last_consolidated_at
  - in_progress
  - source_cursor?
  - lock_ref?
```

推荐标准结果对象：

```text
ConsolidationResult
  - updated_memory_refs[]
  - pruned_memory_refs[]
  - updated_index_ref?
  - summary?
  - status: completed | no_change | failed | killed
```

### 与 canonical object model 的关系

- extraction 输出应收敛到 `DurableMemoryRecord` 或语义等价对象
- commit 结果应返回 durable memory refs，而不是未提交的临时文本
- `ConsolidationResult` 如被跨模块消费，应与 `Session`、`Harness`、`Orchestration` 的事件语义兼容

## 默认实现

当前代码库里，这一层默认由两条链路组成：

- [services/extractMemories/extractMemories.ts](../../cc/services/extractMemories/extractMemories.ts)
  从当前 session transcript 提炼 durable memories
- [services/autoDream/autoDream.ts](../../cc/services/autoDream/autoDream.ts)
  对多个 session 和记忆文件做 consolidation

同时，规范上还应把手动 dream 一并纳入：

- `manual /dream`
  用户显式触发的 consolidation
- `autoDream`
  系统按门槛自动调度的 consolidation

相关默认映射还包括：

- [services/autoDream/consolidationPrompt.ts](../../cc/services/autoDream/consolidationPrompt.ts)
- [services/autoDream/consolidationLock.ts](../../cc/services/autoDream/consolidationLock.ts)
- [memdir/paths.ts](../../cc/memdir/paths.ts)

从默认实现可以看出：

- extraction 是 session-level
- dream / consolidation 是 cross-session
- dream 同时有 manual 和 automatic 两种触发模式
- 两者都不应和 transcript restore 混为同一层

## 状态与恢复语义

`ConsolidationState` 应被视为 durable coordination state，而不是纯进程内标志。

至少应满足：

- `source_cursor`
  可与 `SessionCheckpoint` 或 memory index 语义对齐
- `lock_ref`
  可用于并发去重、crash recovery 或过期判断
- `in_progress`
  不能作为唯一真源；进程崩溃后应允许重建真实状态

如果 consolidation 在运行中断：

- session transcript / checkpoint / resume 不得受损
- 未 commit 的 memory 不得被当作 durable memory 生效
- 下次重试应能基于 `source_cursor` 或等价状态继续，而不是无约束重跑

## 要解决的问题

- 如何把“当前 session 值得记住的内容”从 transcript 中提炼出来
- 如何避免 durable memory 无限制碎片化
- 如何让多个 session 的经验可以后台整合
- 如何让手动 consolidation 与自动 consolidation 共享同一语义模型
- 如何在 consolidation 期间避免并发冲突
- 如何把长期记忆整合与短期 continuity summary 区分开
- 如何保证 consolidation failure / delay 不影响 session restore

## Failure And Delay Semantics

`Memory Consolidation` 是标准扩展，不是 session restore 前提。

因此必须保持：

- extraction 成功但 consolidate 失败时，session 正常继续
- consolidate 成功但 commit 失败时，durable memory view 不得进入部分提交状态
- background consolidation 延迟执行时，recall 可以读到旧版本 memory，但不能读到未提交 memory
- manual `/dream` 与 `autoDream` 必须共享同一成功/失败结果语义

禁止行为：

- 通过改写历史 transcript 模拟 consolidation 成功
- 用 process-local state 假装 memory 已 durable commit
- 将 consolidation 错误上升为 session 不可恢复错误，除非 durable store 本身已损坏且实现明确这样建模

dream 相关的详细规范见 [dream-consolidation.md](dream-consolidation.md)。

## 规范结论

- memory consolidation 应作为 session 域内独立能力存在
- extraction 与 consolidation 应分为两步，不应混成单一操作
- `manual /dream` 与 `autoDream` 应被视为同一 consolidation 能力的两种触发方式
- consolidation 可以后台执行，但状态与锁语义应显式建模
- memory consolidation 不等于 compact，也不等于 verification
- `ConsolidationResult`、`ConsolidationState` 与 `DurableMemoryRecord` 应共同形成可恢复的长期记忆语义
