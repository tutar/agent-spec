# Context Assembly Pipeline

## 职责

本页定义 `Harness` 在一次模型调用前如何装配上下文。

它回答的不是“上下文来自哪里”，而是“这些来源按什么顺序、在哪些平面、以什么治理规则进入最终模型输入”。

## 推荐流水线

```text
1. select bootstrap prompt sections
2. merge structured system_context
3. project transcript / normalized messages
4. inject structured user_context
5. assemble attachments
6. expose capability surface
7. inject per-turn dynamic context
8. apply governance and cache strategy
9. emit final model input
```

## 分阶段语义

### 1. Bootstrap Prompt

先建立 system prompt skeleton。

这一层只负责：

- 稳定行为骨架
- section ordering
- override / append / agent prompt 合成
- static / dynamic boundary

它不应承载 transcript、attachment 或 recall 结果。

### 2. Structured `system_context`

将稳定但非 skeleton 的系统级上下文并入 system 侧。

典型包括：

- repo / environment snapshot
- cache breaker
- runtime guardrail

### 3. Transcript / Normalized Messages

投影 durable transcript 与当前 turn 消息窗口。

这一步处理：

- user/assistant 历史
- tool continuation 相关消息
- compact 后的 continuity window

### 4. Structured `user_context`

把规则文件、日期、回答提醒等 user-facing meta context 注入消息流。

这一步必须晚于 transcript projection，早于 attachments。

### 5. Attachment Assembly

按既定顺序装配 attachment。

此阶段应处理：

- user input attachments
- all-thread attachments
- main-thread-only attachments
- thread-scoped teammate / worker attachments

### 6. Capability Surface

把模型可见工具面并入最终请求。

包括：

- always-loaded tool schemas
- deferred capability surface
- searchable capability exposure

### 7. Per-Turn Dynamic Context

注入按当前轮条件触发的 context。

例如：

- relevant memories
- dynamic skills
- capability delta
- instruction delta

### 8. Governance And Cache Strategy

最后应用：

- budget analysis
- compact / overflow recovery
- long result externalization
- prompt cache boundary placement

这一步不会改变前面各平面的语义归属，只负责控制体积和稳定性。

## 装配原则

### 1. startup-only 与 per-turn 必须分开

- startup-only context
  只在 session-start、agent-start、resume-start、显式 reentry 时注入
- per-turn context
  在每次模型调用前重算

### 2. attachment assembly 是独立阶段

attachment 不应被隐藏在 provider 或 bootstrap prompt 中。

### 3. capability exposure 同时是能力问题和预算问题

deferred/searchable tool surface 的意义不仅是工具发现，也是减少静态 prompt 和 schema 成本。

### 4. compact summary 不是 truth source

compact 只负责 continuity shaping，不替代 transcript、tool result 或 evidence ref。

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

本地 harness 通常 direct-call：

- session transcript
- memory recall
- tool registry
- local persisted result store

### Cloud-Compatible Mapping

云端 harness 仍按同一流水线装配，只是每一步的数据来源可能经远程接口取得。

不允许因为部署在云端而把：

- transcript 投影
- attachment assembly
- prompt cache boundary
- governance decision

改写成另一个行为语义。

## 规范结论

- context engineering 必须按流水线建模，而不是 provider 集合
- bootstrap prompt、structured context、attachments、capability surface、dynamic context 必须分阶段进入模型输入
- startup-only context、per-turn context、governance control 不能混层
