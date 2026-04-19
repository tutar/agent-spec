# Agent Spec

本目录是一套面向多语言 agent 实现者的规范包。


本规范默认面向 `Local-first` 宿主形态：

- 单机运行
- 模块可以 direct-call
- 本地 session / tool / sandbox / harness-managed task 为默认落点

同时要求所有核心接口保持 `Cloud-compatible`：

- transport-neutral
- 可远程化
- 可替换部署位置
- 不把本地实现细节固化为接口前提

本目录按 5 个核心模块组织，每个模块目录都以一个 `README.md` 作为主规范，统一回答四件事：

- 职责
- 稳定接口
- 默认实现
- 要解决的问题

目标不是沉淀某个具体语言或某一版 harness 的实现细节，而是形成一套可向下指导多语言 agent 实现、向上承接不同模型与基础设施能力的稳定接口规范。

## 规范定位

本规范包的主要交付包括：

- 五个核心模块的稳定语义边界
- 跨模块共享的 canonical object model
- `Local-first` 默认语义
- 核心能力与标准扩展能力的分层
- 跨语言 conformance 规则与 golden cases

其中：

- `核心能力`
  宣称实现某个模块时必须保持一致的最小语义
- `标准扩展`
  推荐沉淀的高级能力，如 prompt cache、memory consolidation、background agent；允许采用不同技术方案，但行为语义必须兼容


## 阅读入口

建议先按以下顺序阅读：

1. [module-overview.md](module-overview.md)
2. [terminology-and-ownership.md](terminology-and-ownership.md)
3. 五个核心模块的 `README.md`

## 五个核心模块

- [harness/README.md](harness/README.md)
- [session/README.md](session/README.md)
- [tools/README.md](tools/README.md)
- [sandbox/README.md](sandbox/README.md)
- [orchestration/README.md](orchestration/README.md)

其中 `tools/` 现按职责进一步拆为：

- `tool-model/`
- `command-surface/`
- `builtin/`
- `skills/`
- `mcp/`

其中 [harness/README.md](harness/README.md) 已进一步按多组子主题组织：

- Runtime Core
- Model Provider
- Context Engineering
- Gateway
- Extension And Projection
- Task
- Multi-Agent

在 `Context Engineering` 子主题下，现已将 bootstrap prompt、structured context、attachment assembly、startup context、context governance、prompt cache 统一收敛为一组规范，用于稳定模型可见输入的对象模型与装配流水线。

另外，反思相关能力已按职责拆入现有模块，而不是新增顶层 `reflection` 模块：

- `harness` 下的 verification
- `harness` 下的 post-turn processing
- `session` 下的 memory consolidation

此外，规范现在显式面向两种宿主场景：

- `Local`
  单机部署，模块可 direct-call，本地 session / task / verifier / multi-agent 为主
- `Cloud`
  托管 control plane + remote execution

这两种场景不改变五个核心模块的边界，但会改变它们的部署位置、职责分布和默认实现映射。
其中：

- `Harness`
  表达本地 task、background task、verification、multi-agent 等默认运行时语义
- `Orchestration`
  只表达 cloud 托管控制面与远端可管控编排语义

## 共享文档

- [module-overview.md](module-overview.md)
- [terminology-and-ownership.md](terminology-and-ownership.md)
- [conformance.md](conformance.md)
- [conformance/README.md](conformance/README.md)
- [capability-surface.md](capability-surface.md)
- [extension-packaging.md](extension-packaging.md)
- [object-model.md](object-model.md)
- [schema-serialization.md](schema-serialization.md)
- [test-artifacts.md](test-artifacts.md)
- [orchestration/conformance-scenarios.md](orchestration/conformance-scenarios.md)
- [todo.md](todo.md)

## 设计原则

- 本规范首先稳定模块边界，再允许实现自由演进。
- 优先稳定 `Harness / Session / Tools / Sandbox / Orchestration` 五个对象。
- 优先复用模型原生能力，能力不足时由 harness 补齐。
- 默认以 `Local-first` 方式抽取语义，再要求核心接口保持 `Cloud-compatible`。
- durable history、执行环境、工具编排、托管编排必须解耦。
- 同一套模块边界必须同时适配 Local、Cloud 两种宿主形态。
- 文档中的接口描述是跨语言语义，不是具体语言 API。
- `command` 会作为 `tools` 域内共享抽象出现，但不升级为第六个顶层模块。
- 当前流行优化方案应优先沉淀为标准扩展，而不是直接绑死某一种实现算法。

## 关于实现落点

本规范保留 `Local-first` 的默认宿主落点，但不把任何单一工程结构、类名或实现布局写成规范前提。

实现者可以参考常见 agent runtime 的装配方式来理解模块边界，但合规目标始终是行为语义一致，而不是目录结构一致。

## 关于源码分析材料


如果其他说明文档或 README 引用了当前快照中不存在的分析材料，应将其视为背景说明，而不是规范前提。规范内容必须能够在不依赖缺失文档的前提下独立成立。
