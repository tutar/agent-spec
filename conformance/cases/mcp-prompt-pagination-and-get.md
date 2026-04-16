# Case: MCP Prompt Pagination And Get

## 目标

验证 prompts 的分页列举与 `prompts/get` 语义。

## Preconditions

- server 声明支持 prompts
- prompt 列表可分页

## Ingress

- 分页调用 `prompts/list`
- 对某个 prompt 调用 `prompts/get`

## Expected Runtime Behavior

- page token 被正确消费
- prompt descriptor 与 prompt body 被区分建模
- prompt 被适配到 `Command` 或等价共享对象模型，而不是 `Tool`

## Failure Conditions

- 忽略分页导致列表不完整
- 把 prompt 错误暴露成 tool
