# Tool Model

## 职责

本目录收敛 `Tools` 模块中执行型能力的主线规范。

对 `AgentRuntime` 来说，这一组接口回答的是：

- sampling 前如何装配当前 turn 可见 tool surface
- tool loop 中如何解析、审查并执行 tool call
- tool result、context mutation 与 persisted result ref 如何回流 runtime

## 阅读顺序

1. [tool-definition.md](tool-definition.md)
2. [tool-registry.md](tool-registry.md)
3. [policy-engine.md](policy-engine.md)
4. [../../harness/permission/README.md](../../harness/permission/README.md)
5. [tool-executor.md](tool-executor.md)
6. [tool-streaming-execution.md](tool-streaming-execution.md)

## 边界

- 本目录只覆盖执行型 `tool` 主线
- permission runtime 主规范见 [../../harness/permission/README.md](../../harness/permission/README.md)
- `command` / `review capability` 见 [../command-surface/README.md](../command-surface/README.md)
- coding-oriented baseline 见 [../builtin/README.md](../builtin/README.md)
- `skills`、`mcp`、`plugins` 仍保留各自子目录
