# Case: MCP Sampling With Tools

## 目标

验证 sampling request 在声明支持 `tools` / `toolChoice` 时的语义。

## Preconditions

- client 声明支持 `sampling`
- 若场景包含 `tools`，则 client 也声明支持 sampling 中的 tools

## Ingress

- server 发起 sampling request
- request 包含 messages
- 可选包含 `tools` 与 `tool_choice`

## Expected Runtime Behavior

- sampling request 可进入宿主审批或 policy 审查
- 未声明支持 `tools` 时，不得接受带工具的 sampling request
- 结果与本地 `RequiresAction` / `SdkError` 语义兼容

## Failure Conditions

- 未协商就接受带工具的 sampling request
- sampling 审批链丢失原 `request_id`
