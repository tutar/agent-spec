# Case: MCP Tool Pagination And Call

## 目标

验证 tools 的分页列举与 `tools/call` 语义。

## Preconditions

- server 声明支持 tools
- tool 列表可分页

## Ingress

- 分页调用 `tools/list`
- 对某个 tool 调用 `tools/call`

## Expected Runtime Behavior

- page token 被正确消费
- tool 调用可进入标准 tool lifecycle
- 远端失败与本地 policy / transport failure 被区分

## Failure Conditions

- 忽略分页导致工具集不稳定
- MCP tool 无法参与标准 tool lifecycle
