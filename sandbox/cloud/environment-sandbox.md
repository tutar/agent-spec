# Cloud Environment Sandbox

## 目标

`Cloud Environment Sandbox` 规范 Cloud deployment 下的完整执行底座。

它面向的是更长生命周期的远端执行环境，而不是单次命令包装。

典型场景包括：

- 容器级 agent workspace
- remote execution node
- ephemeral VM
- hosted remote workspace
- VPC-attached execution target
- proxy-mediated external access

## 适用层级

这一层对应：

- container-level isolation
- workspace-level isolation
- remote execution environment
- network boundary
- credential boundary
- proxy / provision lifecycle

它与 `Execution Sandbox` 的区别在于：

- 生命周期更长
- 作用域更大
- 常与 provisioning / orchestration 绑定
- 常承载完整 cloud execution substrate

## 稳定接口

推荐最小接口：

```text
EnvironmentSandbox
  - provision(provision_request) -> sandbox_handle
  - resume(sandbox_handle) -> status
  - inspect(sandbox_handle) -> sandbox_status
  - recycle(sandbox_handle)
  - destroy(sandbox_handle)
```

建议补充：

```text
EnvironmentSandboxStatus
  - sandbox_id
  - isolation_level
  - workspace_ref
  - network_profile
  - filesystem_profile
  - credential_boundary
  - proxy_profile?
  - lifecycle_state
```

## 设计要求

- `Environment Sandbox` 应与 orchestration 的 provisioning 能力协同
- 应允许懒创建、恢复、替换和销毁
- 应把 workspace / image / remote location 视为一等配置
- 不应假设与 harness 共机或同进程
- 应允许把 network boundary、credential boundary、tool proxy 纳入同一环境实例语义
- 应支持 execution target 失效后的 reprovision

## 默认实现映射

当前代码库没有完整、统一的 `Cloud Environment Sandbox` 顶层实现，但存在相关默认映射：

  本地/子进程 session 承载
  agent isolation routing
  remote/teleport 场景下的 workspace/environment 迁移

因此当前仓库更接近：

- `Execution Sandbox` 已较完整
- `Cloud Environment Sandbox` 仅有部分能力散落在 remote/workspace/orchestration 路径中

## 要解决的问题

- 如何为 agent 或 task provision 长生命周期隔离环境
- 如何把容器、VM、remote workspace 统一纳入 sandbox 语义
- 如何与 session/orchestration 配合做恢复和迁移
- 如何把 credential boundary、workspace boundary、network boundary 绑定到环境级实例
- 如何把 proxy、VPC、remote service access 绑定到同一个 cloud execution substrate

## 规范结论

- `Cloud Environment Sandbox` 应作为 Cloud profile 下的完整执行底座存在
- 它不只描述 remote workspace，还应覆盖 network、credential、proxy 与 provision 语义
- 即使当前默认实现不完整，规范上也应保留这一层
