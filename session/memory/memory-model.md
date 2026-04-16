# Session Memory Model

## 目标

本规范定义 `Session` 域中的长短期记忆模型。

这里的 `memory` 不是当前 prompt，也不是 transcript 的别名。规范上应至少区分四类对象：

- `session transcript`
  可恢复的事件历史
- `short-term session memory`
  当前 session 的连续性摘要
- `durable memory`
  跨 session 的可召回知识
- `scoped durable memory`
  durable memory 在 `user / project / agent / local` 维度上的作用域化形态

本规范的目标是稳定以下语义：

- 哪些信息应进入短期记忆
- 哪些信息应进入长期记忆
- 短期记忆、长期记忆与 transcript 的边界
- 记忆的写入、召回、整合与恢复时机
- 记忆如何与 compact、resume、subagent、agent-scoped memory 协同

## 一、核心原则

- transcript 是恢复依据，不是记忆层本身
- 短期记忆服务于 continuity，不服务于跨 session recall
- 长期记忆服务于 recall，不服务于 session restore
- 显式记忆文件属于长期记忆注入面，不属于 session transcript
- agent 私有记忆不是新的记忆类型，而是长期记忆的作用域变体

## 二、分层模型

### 1. Session Transcript

`SessionTranscript` 是会话的可恢复历史。

它负责：

- 保存原始 turn、tool use、tool result、system event
- 支撑 resume、replay、audit、restore
- 为短期记忆和长期记忆提供提炼源

它不负责：

- 作为 continuity summary
- 直接作为长期知识库
- 直接决定 recall 结果

### 2. Short-Term Session Memory

`ShortTermSessionMemory` 是当前 session 的 continuity object。

它负责：

- 维护当前会话的压缩连续性表示
- 为 compact 提供 continuity summary
- 为 resume 后的快速继续提供低成本状态
- 为长任务 handoff 提供会话级摘要

它不负责：

- 替代 transcript
- 承担跨 session recall
- 作为 durable knowledge 的存储

应进入短期记忆的信息包括：

- 当前任务目标与阶段性进展
- 尚未完成的中间结论
- 最近关键决策、约束和风险
- compact 后仍需保留的局部上下文

不应进入短期记忆的信息包括：

- 大量原始 stdout / file dump
- 可从 transcript 精确重放的细粒度历史
- 通用用户偏好或项目长期知识

### 3. Durable Memory

`DurableMemory` 是跨 session 的可召回知识层。

它负责：

- 保存用户偏好、项目知识、约束、经验、gotchas
- 保存对未来回合可能有价值的抽象信息
- 为后续 session 提供 bounded recall 候选

它不负责：

- 替代 session restore
- 保存每轮完整对话
- 充当当前 turn 的 working state

应进入长期记忆的信息包括：

- 稳定的用户偏好
- 项目级规则与约束
- 重复出现的问题及解决方式
- API / 环境 / workflow 的注意事项
- 值得跨会话复用的设计决策

不应进入长期记忆的信息包括：

- 仅对当前 turn 有价值的临时状态
- 纯粹可由 transcript 重新推导的细节
- 已经过时且没有保留价值的操作痕迹

### 4. Scoped Durable Memory

`ScopedDurableMemory` 是 durable memory 的作用域化形态。

推荐至少支持：

- `user`
  对所有项目通用
- `project`
  在当前项目内共享
- `agent`
  某类 agent 的专用长期知识
- `local`
  仅当前环境或当前机器有效

规范上，`agent-scoped memory` 属于这一层，而不是单独的新 memory 类型。

### 5. Durable Memory Injection

`DurableMemoryInjection` 指将长期记忆显式注入当前 runtime 的机制。

它包括但不限于：

- 项目记忆文件
- 用户记忆文件
- agent 记忆入口文件
- 其他可直接作为 context source 的 durable memory 表达

这一层的职责是“注入”，不是“召回”或“恢复”。

## 三、生命周期

### 1. 短期记忆生命周期

推荐流程：

1. session 进行中积累 transcript
2. 到达阈值或自然停顿点
3. 从 transcript 增量提炼 continuity summary
4. 更新 short-term memory
5. compact / resume / away summary 消费该摘要

短期记忆更新应满足：

- 基于增量，而不是每次全量重算
- 在安全边界更新，避免截断未闭合的 tool chain
- 允许异步更新，但 compact 前应支持等待

### 2. 长期记忆生命周期

推荐流程：

1. 完整 turn 或 query loop 结束
2. 从 transcript 中提炼 durable memory 候选
3. 写入 durable memory store
4. 周期性执行 consolidate / cleanup
5. 后续 turn 通过 recall 取回相关 memory

长期记忆更新应满足：

- 以完整语义单元为边界，而不是任意中间状态
- 支持后台提炼
- 支持去重、合并、整合
- 支持 freshness / source metadata

### 3. 召回生命周期

推荐流程：

1. turn 开始前进行 memory recall prefetch
2. 基于 query、runtime context、tool context 选择候选
3. 对 recall 结果做数量与体积约束
4. 将 recall 结果作为 attachments / context sources 注入当前 turn

召回必须满足：

- bounded
- 可去重
- 可过滤已注入对象
- 不要求把整个 memory store 暴露给模型

## 四、与其他模块的边界

### 1. 与 Session 的边界

`Session` 管理：

- transcript
- runtime state
- restore
- short-term memory linkage

`Session` 不直接定义 durable memory store 的内部实现。

### 2. 与 Harness 的边界

`Harness` 负责：

- 决定何时触发记忆读写
- 将短期记忆和长期记忆接入当前 turn
- 在 compact / resume 时消费短期记忆

`Harness` 不应把 transcript 和 memory 混成一个对象。

### 3. 与 ContextProvider 的边界

`ContextProvider` 负责：

- 将 durable memory injection 变成当前 turn 的上下文输入

它不负责：

- durable memory 的提炼
- session transcript 的恢复

### 4. 与 ContextGovernance 的边界

`ContextGovernance` 负责：

- token budget
- compact 触发
- pruning 策略

它可以消费短期记忆，但不拥有短期记忆。

## 五、推荐最小接口

```text
ShortTermMemoryStore
  - load(session_id) -> short_term_memory | null
  - update(session_id, transcript_delta, current_memory) -> updated_memory
  - get_coverage_boundary(session_id) -> transcript_cursor | null
  - wait_until_stable(session_id, timeout_ms) -> ready | timed_out
```

```text
DurableMemoryStore
  - put(memory_record) -> memory_id
  - update(memory_id, patch) -> memory_record
  - delete(memory_id) -> deleted
  - list(selector) -> memory_index
  - read(memory_refs) -> memory_payloads
```

```text
DurableMemoryExtractor
  - extract(transcript_slice, existing_memory_context) -> memory_records
```

推荐 `memory_record` 至少包括：

```text
DurableMemoryRecord
  - memory_id?
  - scope
  - summary
  - source_session_id?
  - source_event_refs?
  - freshness?
  - metadata?
```

## 六、失败与延迟语义

`memory consolidation` 作为标准扩展，可以异步、延迟或后台执行，但必须满足：

- consolidation 失败不得破坏 transcript / checkpoint / resume
- recall 若读取到旧版本 durable memory，行为上允许暂时滞后，但不能造成 session restore 语义漂移
- 去重、合并、重写 durable memory 时，必须保留可追溯 source metadata 或语义等价引用
- consolidation 结果应体现在 durable memory store，而不是通过改写历史 transcript 模拟

## 七、规范结论

- `DurableMemoryRecord` 应作为 durable memory 的最小共享对象
- memory consolidation 是标准扩展，不是 session restore 的前提
- consolidation 可以失败或延迟，但不能让 session 丢失可恢复性

```text
MemoryRecallEngine
  - prefetch(query, runtime_context) -> recall_handle
  - collect(recall_handle) -> memory_attachments
  - dedupe(memory_attachments, already_loaded) -> filtered_attachments
```

```text
MemoryConsolidator
  - schedule(selector) -> consolidation_job
  - run(consolidation_job) -> consolidation_result
```

推荐的 `memory_record` 最小字段：

- `memory_id`
- `scope`
- `type`
- `title`
- `summary`
- `content`
- `source`
- `created_at`
- `updated_at`
- `freshness`

## 六、推荐默认策略

### 短期记忆

- 以会话连续性为第一目标，而不是信息完备性
- 在上下文增长达到阈值后再更新
- 优先在自然停顿点或 tool 链闭合后更新
- compact 发生时优先读取最新稳定版本

### 长期记忆

- 只提炼未来可复用的信息
- 写入前做去重或合并判断
- 召回时优先选择少量高确定性的条目
- 周期性做 consolidate，避免 memory corpus 无界增长

### 作用域策略

- 用户偏好优先写入 `user`
- 项目约束优先写入 `project`
- agent 专用知识优先写入 `agent`
- 机器/环境特定信息优先写入 `local`

## 七、默认实现映射

本仓库当前可映射出一套符合该规范的默认实现：

### Short-Term Session Memory

- [services/SessionMemory/sessionMemory.ts](../../cc/services/SessionMemory/sessionMemory.ts)
- [services/SessionMemory/sessionMemoryUtils.ts](../../cc/services/SessionMemory/sessionMemoryUtils.ts)

默认实现特征：

- 只在主会话线程运行
- 由 post-sampling hook 触发
- 以 token 增长和 tool use 阈值决定是否更新
- 在自然停顿点或 tool use 安全边界更新
- 生成的是 continuity summary，而不是 transcript 副本
- compact 前支持等待进行中的更新完成

### Durable Memory

- [services/extractMemories/extractMemories.ts](../../cc/services/extractMemories/extractMemories.ts)
- [memdir/findRelevantMemories.ts](../../cc/memdir/findRelevantMemories.ts)
- [services/autoDream/autoDream.ts](../../cc/services/autoDream/autoDream.ts)

默认实现特征：

- 在完整 query loop 结束后后台提炼 durable memory
- recall 先基于 memory manifest/header 选候选，再做有界注入
- 支持跨 session consolidation
- durable memory 与 transcript restore 分离

### Durable Memory Injection

- [context.ts](../../cc/context.ts)
- [utils/claudemd.ts](../../cc/utils/claudemd.ts)

默认实现特征：

- 项目规则、用户规则和显式记忆文件通过 context assembly 注入
- 注入链路与 transcript restore 分离

### Scoped Durable Memory

- [tools/AgentTool/agentMemory.ts](../../cc/tools/AgentTool/agentMemory.ts)

默认实现特征：

- 为 agent 提供 `user / project / local` 作用域的专用 durable memory
- 以 agent 类型作为记忆命名空间

## 八、规范结论

- `short-term session memory`、`durable memory`、`scoped durable memory` 必须明确分层
- transcript、compact、recall、restore 不应共享同一个 memory 抽象
- 短期记忆面向 continuity，长期记忆面向 recall
- 当前仓库可作为一套默认实现映射，但不限定规范必须采用同样的存储介质或触发机制
