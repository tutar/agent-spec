# Context Engineering

## 职责

`ContextEngineering` 定义 `Harness` 如何把来自 `Session`、`Tools`、`Sandbox` 与当前交互输入的材料，组织成一次模型调用可见的上下文。

它负责：

- 定义模型可见输入的对象模型
- 定义 bootstrap prompt、structured context、attachments、capability surface 的边界
- 定义上下文装配流水线
- 定义 startup、turn-zero、resume、agent-start 的注入差异
- 定义 context budget、compact、externalization、cache-aware assembly 等治理策略

它不负责：

- durable transcript 持久化
- tool 执行本身
- sandbox enforcement
- verifier / task lifecycle

## 设计定位

本子域采用 `Local-first + Cloud-compatible`：

- `Local-first`
  默认实现映射来自本地 harness 与本地 session/tool/sandbox 的协作路径。
- `Cloud-compatible`
  在云端形态下，通常是整套 harness 作为无状态服务运行；`ContextEngineering` 仍在 harness 内部执行，只是它消费的 transcript、memory、tool surface、evidence refs、environment context 来自远程模块接口，而不是本地直读。

这意味着：

- `ContextEngineering` 始终属于 `Harness`
- 它不是独立顶层模块
- 它也不是一个单独远程服务

## 要解决的问题

- 模型可见输入不只是一个 prompt 字符串，规范必须明确它的多平面结构
- 稳定 prompt skeleton 与高波动上下文必须分开
- startup-only context、turn-zero briefing、resume context 不能混成一类
- attachment、memory recall、tool schemas、request metadata 都会影响模型输入，但来源和预算策略不同
- 长会话中必须避免 context drift、evidence loss 和无意义 cache bust

## 子主题

- [bootstrap-prompts.md](bootstrap-prompts.md)
  定义 system prompt skeleton、section registry、静态/动态边界与覆盖关系。
- [context-input-model.md](context-input-model.md)
  定义模型可见输入的最小对象模型与 context planes。
- [context-assembly-pipeline.md](context-assembly-pipeline.md)
  定义从 transcript、structured context、attachments、tool surface 到最终模型输入的装配流水线。
- [context-provider.md](context-provider.md)
  只定义 provider 接口、作用域、生命周期、cacheability 与 provenance。
- [attachment-assembly.md](attachment-assembly.md)
  定义 attachment 的来源、顺序、thread scope 与 audience。
- [startup-and-turn-zero-context.md](startup-and-turn-zero-context.md)
  定义 session-start、agent-start、turn-zero、resume-start 的差异。
- [context-governance.md](context-governance.md)
  定义 context budget、compact、externalization、delta injection 与 drift mitigation。
- [prompt-cache-strategy.md](prompt-cache-strategy.md)
  定义 stable prefix、dynamic suffix、cache boundary 与 cache break 语义。

## 规范结论

- 上下文工程是 `Harness` 的内部能力，不是独立模块
- 上下文工程不等于“拼 prompt”，而是多平面输入装配
- bootstrap prompt、structured context、attachments、capability surface、memory recall 必须分层
- 本地与云端的差异只应体现在依赖输入的获取方式，不应改变装配语义
