# Attachment Assembly

## 职责

`AttachmentAssembly` 定义 message-level context 如何进入模型输入。

它覆盖的不只是用户上传文件，还包括各种运行时附加上下文。

## 最小对象

```text
AttachmentEnvelope
  - source
  - audience
  - thread_scope
  - ordering_class
  - payload_ref | inline_payload
  - provenance
```

## 推荐分组

- `user_input`
  用户在当前输入中显式附加的文件、资源、图片、mentions。
- `all_thread`
  主线程与 worker/subagent 都可能需要的运行时附加上下文。
- `main_thread_only`
  只对主线程或 leader 有意义的附加上下文。
- `agent_scoped`
  只对某个特定 worker、teammate 或 agent task 可见的附加上下文。

## 装配顺序

推荐稳定顺序：

1. `user_input`
2. `all_thread`
3. `main_thread_only`
4. `agent_scoped`

实现可以在类内再细分，但不应把 attachment 当作无序集合。

## 语义要求

### 1. attachment 必须支持 thread scope

不是所有 attachment 都应被所有线程看到。

至少要支持：

- main session only
- all threads
- single worker / single agent scope

### 2. attachment 必须支持 audience

同样一条 payload 可能面向：

- model-visible message projection
- debug / audit only
- UI-only projection

不要把所有附加信息都直接送给模型。

### 3. attachment 应优先使用引用而非全量内联

大文件、长工具结果、诊断输出、task output 在多数情况下应：

- 给模型一个 preview 或摘要
- 保留 `payload_ref`
- 必要时按引用二次读取

### 4. attachment 与 transcript 必须区分

attachment 是运行时装配面，不等于 durable transcript 本体。

它可以被投影进消息流，但其来源、可见性和治理策略独立于 transcript。

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

- `payload_ref` 常映射到本地文件、工具结果持久化路径、本地资源句柄
- thread scope 常来自本地 runtime state、agent task state 或当前 UI 视图状态

### Cloud-Compatible Mapping

- `payload_ref` 可以是远程对象、资源句柄、blob ref
- attachment ordering、scope 和 audience 语义必须保持不变

## 规范结论

- attachment 是一等上下文平面
- attachment 必须有顺序、scope、audience 和 provenance
- attachment 不应被建模成无序文件列表
