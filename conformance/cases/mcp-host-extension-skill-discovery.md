# Case: MCP Host Extension Skill Discovery

## 目标

验证 host-specific `mcp skill` 发现语义，同时确保其不被误当成 MCP core requirement。

## Preconditions

- host 明确声明支持 `mcp skill` extension
- server 暴露符合 host 约定的 `skill://` resource 或等价 metadata

## Ingress

- client 列举 resources
- host 执行 `mcp skill` 发现逻辑

## Expected Runtime Behavior

- `mcp skill` 与普通 MCP prompt 明确区分
- 未声明支持该 extension 的实现仍可宣称 MCP core compatibility

## Failure Conditions

- 把 `mcp skill` 发现上升为所有 MCP client 的核心要求
- 无法区分 `mcp skill` 与普通 MCP prompt / resource
