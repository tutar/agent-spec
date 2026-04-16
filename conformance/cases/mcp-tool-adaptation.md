# Case: MCP Tool Adaptation

## 目标

验证 MCP 接入后的能力映射语义。

## Preconditions

- runtime 支持 MCP
- 至少存在一个 MCP server

## Ingress

- 枚举 MCP 暴露的能力
- 触发一次 MCP tool 调用
- 若实现支持 prompt / skill 映射，也同时检查枚举结果

## Expected Mapping

必须保持以下角色区分：

- MCP tools -> `Tool`
- MCP prompts -> `Command` 或等价共享对象模型
- host-specific `mcp skill` extension 如存在，才映射到 `Skill` 语义对象

## Expected Runtime Behavior

- MCP tool 可通过标准 tool execution 语义调用
- MCP prompt 不应被错误暴露成 tool
- 若宿主支持 `mcp skill` extension，该扩展不应被错误退化成普通 prompt 列表项

## Allowed Variance

- 不同语言实现可使用不同 MCP client
- Local host 可增加 bundle / install / trust 流程
- 不支持 `mcp skill` extension 的实现仍可宣称 MCP core compatibility

## Failure Conditions

- 把 MCP prompt 映射成 tool
- 把 host-specific `mcp skill` extension 误写成 core MCP requirement
- MCP tool 无法参与标准 tool lifecycle
