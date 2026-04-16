# Command Surface

## 职责

本目录收敛 `Tools` 模块中非直接执行型入口对象的规范。

它主要覆盖：

- `Command` 共享对象模型
- prompt / local / review capability 的入口语义
- command surface 与 tool surface 的边界
- command-like capability 与 orchestration 的桥接

## 阅读顺序

建议按以下顺序阅读：

1. [command-model.md](command-model.md)
2. [reflection-and-verification-commands.md](reflection-and-verification-commands.md)

## 边界

- 本目录不定义执行型 tool 的 schema / policy / executor 主线
- 执行型工具规范见 [../tool-model/README.md](../tool-model/README.md)
- `skills`、`mcp` 仍保留各自子目录
