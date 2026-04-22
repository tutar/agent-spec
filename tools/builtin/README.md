# Builtin Baseline

## 职责

本目录定义 coding-oriented host 推荐保留的 built-in capability baseline。

对 `AgentRuntime` 来说，这一组文档回答的是：

- 默认至少应装配哪一组 builtin capabilities
- 哪些 builtin family 构成 coding/runtime baseline
- baseline capability surface 与完整 tool registry 的边界是什么

## 目录

- [builtin-tool-baseline.md](builtin-tool-baseline.md)

## 边界

- 本目录是 baseline catalog，不是完整工具注册表
- 执行型工具主线见 [../tool-model/README.md](../tool-model/README.md)
- plugin- or protocol-contributed capabilities 不在本目录建模
