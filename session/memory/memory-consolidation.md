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

## 要解决的问题

- 如何把“当前 session 值得记住的内容”从 transcript 中提炼出来
- 如何避免 durable memory 无限制碎片化
- 如何让多个 session 的经验可以后台整合
- 如何让手动 consolidation 与自动 consolidation 共享同一语义模型
- 如何在 consolidation 期间避免并发冲突
- 如何把长期记忆整合与短期 continuity summary 区分开

dream 相关的详细规范见 [dream-consolidation.md](dream-consolidation.md)。

## 规范结论

- memory consolidation 应作为 session 域内独立能力存在
- extraction 与 consolidation 应分为两步，不应混成单一操作
- `manual /dream` 与 `autoDream` 应被视为同一 consolidation 能力的两种触发方式
- consolidation 可以后台执行，但状态与锁语义应显式建模
- memory consolidation 不等于 compact，也不等于 verification
