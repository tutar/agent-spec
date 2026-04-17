# Security Boundary

## 目标

安全边界规范定义控制面、执行面和凭据面的分离方式。

其核心目标不是“提醒模型小心”，而是结构性地限制模型生成代码、prompt injection 或工具滥用带来的横向提权风险。

## 核心分层

推荐至少区分三层：

- `Control Plane`
  session、harness、policy、orchestration
- `Execution Plane`
  sandbox、shell、code runner、device、browser automation
- `Credential Plane`
  vault、resource-bound auth、OAuth broker、tool proxy

## 结构性原则

- 执行面默认不可信
- 控制面不应把长生命周期凭据暴露给执行面
- 凭据访问应最小化、可审计、可撤销
- prompt injection 不应被视为仅靠提示词就能缓解的问题

## 推荐认证模式

### 1. Resource-Bound Auth

凭据在资源初始化时就被绑定到资源，而不是交给 agent。

例如：

- repo clone 时把 git remote 凭据接到资源上
- object store mount 时由平台完成挂载

agent 只看到可操作资源，不直接看到 token。

### 2. Vault-Backed Auth

凭据存放在 sandbox 之外的 vault 中。

agent 通过受控 proxy 使用某个能力，proxy 再去取凭据并执行外部调用。

agent 不直接获得原始密钥。

## Tool Proxy 模型

对于 MCP、SaaS API、外部 webhook 等能力，推荐加入 tool proxy：

```text
Harness
  -> Tool Proxy
  -> Vault / Credential Broker
  -> External Service
```

proxy 负责：

- 凭据查找
- 最小权限调用
- 日志与审计
- 输出过滤或脱敏

## 不推荐做法

- 直接把 OAuth token 注入 sandbox 环境变量
- 让模型生成代码读取本地密钥文件
- 让 harness 与 sandbox 共用同一高权限运行时
- 用“模型不会这么做”作为安全前提

## 与 PolicyEngine 的边界

- `PolicyEngine`
  决定某个动作是否允许发生
- `Security Boundary`
  决定即使动作发生，模型能接触到哪些敏感面

两者互补，不能互相替代。

## 与 Sandbox 的边界

sandbox 可以执行受限动作，但不应成为凭据持有者。

若某些资源必须在 sandbox 内可用，也应通过短期、最小权限、可审计机制提供，而不是直接暴露长期密钥。

## 事件与审计要求

推荐至少记录：

- 哪个 session 请求了哪类敏感能力
- 是否经过 proxy
- 使用了哪种 credential source
- 是否发生凭据升级或失败

这些日志应归于 control plane，而不是只存在于 sandbox 本地。

## 规范结论

- 凭据、控制、执行三者必须结构性分离
- 安全边界是架构问题，不是 prompt 问题
- 所有语言 agent 实现都应把凭据隔离视为顶层规范，而不是部署细节
