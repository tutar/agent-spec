# Local Environment Sandbox

## 目标

`Local Environment Sandbox` 描述单机部署下的环境级边界。

它的核心不是远端 provision，而是：

- 给定目录范围内的访问权
- 本地 workspace root
- 对本地文件系统可见范围的限制

它通常作为 `Execution Sandbox` 之外的附加边界存在，而不是 `Local` profile 的默认主路径。

## 核心语义

`Local Environment Sandbox` 应优先表达目录授权，而不是容器/VM 编排。

最小要求：

- agent 只能访问给定目录树
- 应显式声明 `workspace_root`
- 应显式声明允许读写的路径范围
- 不应假设可以读取任意本地目录、任意环境变量或任意宿主资源

## 推荐标准对象

```text
LocalEnvironmentBoundary
  - boundary_id
  - workspace_root
  - allow_read_paths
  - allow_write_paths
  - deny_paths?
  - visible_env_vars?
```

## 设计要求

- 重点是 filesystem boundary，不是 remote provisioning
- 可以由 host-native directory boundary、本地 isolated workspace 或等价机制实现
- 不要求完整 `provision / reprovision / wake-resume` 生命周期
- 若实现提供本地隔离工作区，也应先保持“目录授权”语义，再考虑更重的隔离载体

## 与 Execution Sandbox 的关系

- `Execution Sandbox`
  负责 per-command / per-execution 限制
- `Local Environment Sandbox`
  负责本地可见目录树与工作区边界

两者可以并存，但不应混成一层。

## 默认实现映射

当前代码库没有独立、完整的 `Local Environment Sandbox` 顶层实现。

更接近的默认落点是：

  把本地文件系统与权限限制投影到执行配置
  在命令执行链中消费这些限制

这意味着当前默认实现仍以 `Execution Sandbox` 为主，而本地环境级边界更多通过路径和权限配置表达。

## 规范结论

- `Local Environment Sandbox` 应被收窄为目录授权型环境边界
- 它不是 Cloud 那种完整 execution substrate
- 对 `Local` 来说，最重要的环境级语义是“只允许访问给定目录树”
