# Command Model

## 职责

`Command` 是 `tools` 域内的共享对象模型。

它不是第六个顶层模块，也不是对 `Tool` 的替代。它的作用是为默认实现提供一个统一承载层，用来表达：

- prompt command
- local command
- local-jsx command
- skill 的默认运行时对象
- mcp prompt 的默认运行时对象
- reflection / verification 这类高阶 review capability 的入口对象

换句话说：

- `Tool` 是稳定能力接口
- `Skill` 是稳定能力语义接口
- `Command` 是默认实现中承载部分能力的共享对象模型

## 稳定接口

规范层不要求所有语言 agent 实现都暴露名为 `Command` 的公共类型，但要求保留等价语义。

推荐最小语义：

```text
Command
  - id / name
  - kind
  - description
  - visibility
  - source
  - metadata
  - invoke()
```

其中 `kind` 至少应允许表达：

- `prompt`
- `local`
- `local_ui`
- `review`

如果某个语言实现不公开 `Command` 类型，也应在内部保留等价区分，避免把 prompt/workflow 型对象直接压成 `Tool`。

对于 `review` 一类 command，`invoke()` 的默认后端可以不是本地函数，而是 orchestration 中的 subagent / task。

## 默认实现

当前代码库中的默认对象模型在：

  定义 `PromptCommand`、`LocalCommand`、`LocalJSXCommand` 及统一 `Command`
  负责命令装配、筛选和对模型可见性的整理

源码里的关键事实是：

- `Skill` 默认由 `Command(type='prompt')` 承载
- `mcp prompts` 默认也由 `Command(type='prompt')` 承载
- `Command` 比 `Skill` 更宽，比 `Tool` 更偏 prompt/entrypoint 侧
- 某些 command 的执行可以委托给 orchestration，而不是直接走 `ToolExecutor`

## 要解决的问题

- 如何解释源码里 `skill` 与 `command` 的关系
- 如何在不新增顶层模块的前提下表达 prompt/workflow 型对象
- 如何避免把所有外部能力都压进 `Tool`
- 如何为 MCP prompts、skills、本地命令提供统一对象承载层
- 如何表达“上层像 command、下层靠 verifier task 执行”的能力

## 与 Tool / Skill 的关系

- `Tool`
  执行型能力对象，强调 schema、权限、调用和并发
- `Skill`
  能力语义对象，强调发现、加载、激活和桥接调用
- `Command`
  默认实现里的共享载体，承载 prompt/local/ui 型入口对象

默认映射关系建议明确为：

```text
tool         -> Tool
skill        -> Skill (default carrier may be Command)
mcp prompt   -> Command
local slash  -> Command
review flow  -> Command (may delegate to Orchestration)
```

## Exposure Boundary

为了避免不同 surface 混淆，推荐至少保持以下边界：

- `Tool`
  可以直接暴露给模型进行 schema-based invocation
- `Skill`
  默认先作为 skill / command 暴露；是否桥接成 tool 由宿主决定
- `MCP prompt`
  默认属于 command surface，不应自动伪装成 tool
- `MCP resource`
  默认不属于 tool；若要暴露给模型，通常先经过 adapter / command / context provider
- `local command`
  默认不属于 tool，除非宿主显式桥接

若某项能力被桥接到多个 surface：

- 原始来源必须可追溯
- visibility / policy / invoke path 仍需保留差异

## 当前源码映射

  通过 `createSkillCommand()` 构造 skill 对应的 command
  通过 `fetchCommandsForClient()` 构造 mcp prompt 对应的 command
  把 command 形态的 skill 暴露给模型调用

## 规范结论

- `Command` 应进入 spec，但仅作为 `tools` 域内共享抽象
- `Command` 不应升级为新的顶层模块
- command-like capability 可以委托 orchestration 执行，但其入口语义仍属于 `Tools`
- 各语言 agent 实现可以不用复刻当前源码的命名，但必须保留等价语义分层
- agent 不应因为统一 command surface 而抹平 skill / mcp prompt / local command / review capability 的来源差异
