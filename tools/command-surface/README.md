# Command Surface

## 职责

本目录收敛 `Tools` 模块中非直接执行型入口对象的规范。

对 `AgentRuntime` 来说，这一组接口回答的是：

- turn entry / user input routing 时如何识别 command-like capability
- prompt / local / review 型入口对象如何进入当前 interaction
- 哪些 capability surface 不应被压成执行型 tool

## 阅读顺序

1. [command-model.md](command-model.md)
2. [reflection-and-verification-commands.md](reflection-and-verification-commands.md)

## 边界

- 本目录不定义执行型 tool 的 schema / policy / executor 主线
- 执行型工具规范见 [../tool-model/README.md](../tool-model/README.md)
- `skills`、`mcp`、`plugins` 仍保留各自子目录
