# Case: MCP Resource Subscribe And List Changed

## 目标

验证 resources 的 subscribe / updated / list_changed 观察语义。

## Preconditions

- server 声明支持 resources 与 subscribe

## Ingress

- client 列举 resources
- client 对某个 resource 发起 subscribe
- server 发出 `notifications/resources/updated`
- server 发出 `notifications/resources/list_changed`

## Expected Runtime Behavior

- resource `uri` 保持稳定身份语义
- `updated` 与 `list_changed` 被区分处理
- 观察面变化不会错误投影成 tool result

## Failure Conditions

- `updated` 与 `list_changed` 语义混淆
- 订阅后的变化未能到达本地观察面
