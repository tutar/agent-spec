# Case: MCP Auth Discovery And Scope Upgrade

## 目标

验证 HTTP transport 下 authorization discovery、token 获取与 `WWW-Authenticate` scope upgrade 语义。

## Preconditions

- runtime 支持 MCP HTTP transport
- server 需要 OAuth/OIDC authorization

## Ingress

- client 执行 auth discovery
- 获取初始 token
- server 返回带额外 scope 的 `WWW-Authenticate`
- client 完成 scope upgrade 并重试

## Expected Runtime Behavior

- auth discovery 与 session continuity 分离
- scope upgrade 不会伪装成全新 MCP server
- 失败时产生明确的 auth / transport error

## Failure Conditions

- 把 `stdio` 错误纳入 HTTP auth 逻辑
- 忽略 `WWW-Authenticate` challenge
- token refresh 或 scope upgrade 导致 server identity 漂移
