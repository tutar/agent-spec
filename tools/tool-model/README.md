# Tool Model

## 职责

本目录收敛 `Tools` 模块中执行型能力的主线规范。

这里回答的是：

- tool 如何被定义
- tool 如何被注册与解析
- tool 如何被授权
- tool 如何被执行
- tool 如何以 batch / streaming 模式发射结果
- tool 的大结果、context mutation 与恢复语义如何闭合

## 阅读顺序

建议按以下顺序阅读：

1. [tool-definition.md](tool-definition.md)
2. [tool-registry.md](tool-registry.md)
3. [policy-engine.md](policy-engine.md)
4. [tool-executor.md](tool-executor.md)
5. [tool-streaming-execution.md](tool-streaming-execution.md)

## 边界

- 本目录只覆盖执行型 `tool` 主线
- `command` / `review capability` 见 [../command-surface/README.md](../command-surface/README.md)
- coding-oriented baseline 工具集合见 [../builtin/README.md](../builtin/README.md)
- `skills`、`mcp` 仍保留各自子目录
