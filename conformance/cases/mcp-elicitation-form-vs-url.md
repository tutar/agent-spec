# Case: MCP Elicitation Form Vs URL

## 目标

验证 elicitation 的 `form` 与 `url` 两种模式的分流语义。

## Preconditions

- client 声明支持 `elicitation`

## Ingress

- 一次发起 `form` mode elicitation
- 一次发起 `url` mode elicitation

## Expected Runtime Behavior

- `form` mode 走结构化输入面
- `url` mode 走外部受控页面或等价安全入口
- 敏感信息请求优先使用 `url` mode

## Failure Conditions

- 两种模式被压成同一普通文本输入
- 敏感信息仍通过普通 `form` mode 收集
