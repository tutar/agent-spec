# Sandbox Capability Model

## 目标

本文件定义 `sandbox` 模块的统一能力模型。

规范上，`sandbox` 不应只被理解成某一种具体实现，例如：

- 容器
- 进程包裹
- 远端 workspace

而应首先被定义成：

`对执行面施加隔离与限制的能力模型`

## 核心能力维度

建议至少建模以下能力维度：

### 1. Filesystem

- `allow_read_paths`
- `deny_read_paths`
- `allow_write_paths`
- `deny_write_paths`
- `workspace_roots`

### 2. Network

- `allowed_domains`
- `denied_domains`
- `allow_unix_sockets`
- `allow_local_binding`
- `proxy_config`

### 3. Process

- `allow_process_spawn`
- `allow_nested_execution`
- `allow_interpreters`
- `resource_limits`

### 4. Environment

- `cwd`
- `env_policy`
- `visible_env_vars`
- `redacted_env_vars`

### 5. Device / Host Access

- `allow_browser`
- `allow_gui`
- `allow_local_devices`
- `allow_host_integration`

## 标准对象

推荐共享对象：

```text
SandboxCapabilityProfile
  - profile_id
  - filesystem
  - network
  - process
  - environment
  - device_access

SandboxIsolationLevel
  - none
  - process
  - workspace
  - container
  - vm
  - remote_workspace
```

## 设计要求

- 能力模型应先于实现模型定义
- 同一 sandbox 接口应允许不同 isolation level 实现
- tools、policy、orchestration 应消费 capability profile，而不是依赖具体后端名字

## 默认实现映射

当前代码库的默认实现主要围绕：

- filesystem restrictions
- network restrictions
- process-level wrapping


## 规范结论

- `sandbox` 首先是能力模型，其次才是具体执行后端
- 容器级与进程级实现都应映射到同一 capability model
