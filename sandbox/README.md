# Sandbox And Execution

## 职责

`Sandbox` 模块负责定义执行面的隔离与限制。

规范上需要明确区分两层：

- `Execution Sandbox`
  单次工具执行级别的 OS/进程级沙箱
- `Environment Sandbox`
  按 `Local / Cloud` 分化的长期环境级隔离边界

此外，还需要有统一的 capability model，把不同实现映射到同一语义平面。

不同宿主形态下，这两层的重心通常不同：

- `Local`
  以 `Execution Sandbox` 为主，必要时也可叠加 `Environment Sandbox`
- `Cloud`
  常以 `Environment Sandbox` 为主，`Execution Sandbox` 为辅

## 要解决的问题

- 如何把进程级与容器级 sandbox 放进同一规范
- 如何让 tool execution level sandbox 与 environment-level sandbox 共存
- 如何在执行面失效时不丢失 session
- 如何把不同执行后端统一映射到稳定 capability model
- 如何让 Local、Cloud 在不同隔离实现上仍保持同一 sandbox 语义

## 核心原则

- sandbox 规范应先定义能力模型，再允许多种实现
- `Execution Sandbox` 与 `Environment Sandbox` 必须分层
- harness 不应与某一种 sandbox 实现耦死
- sandbox 失败应被表达为执行面失败，而不是 session 消失

## 规范分层

### 1. Capability Model

统一定义 sandbox 可限制的能力：

- filesystem
- network
- process
- environment
- device / host access

### 2. Execution Sandbox

定义单次 tool execution 的包裹式沙箱：

- command wrapping
- per-execution restriction
- violation reporting

### 3. Environment Sandbox

定义按 profile 分化的环境级沙箱：

- `Local`
  目录授权与 workspace root 边界
- `Cloud`
  remote workspace / container / VM / proxy / credential / provision 底座

## 默认实现

当前代码库中的默认实现主要落在 `Execution Sandbox`：

  作为默认 shell execution entry
  作为外部 sandbox runtime 的适配层

它主要体现的是：

- OS/进程级执行沙箱
- command wrapping
- filesystem/network policy 投影

环境级 sandbox 在当前仓库中只有部分映射，更多散落在 remote/workspace/orchestration 路径里。

## 与其它模块的边界

- `ToolExecutor`
  负责编排调用顺序和取消，不替代 sandbox
- `PermissionSystem`
  消费 working-directory、filesystem allowlist 与 escalation 输入，但不等于 sandbox
- `Session`
  保存 durable state，不等于执行环境
- `Orchestration`
  负责 provision / route / recover，可消费 environment sandbox
- `Security Boundary`
  负责控制面、执行面、凭据面的结构性隔离

## 目录内文档

- [capability-model.md](capability-model.md)
- [execution-sandbox.md](execution-sandbox.md)
- [local/README.md](local/README.md)
- [local/environment-sandbox.md](local/environment-sandbox.md)
- [cloud/environment-sandbox.md](cloud/environment-sandbox.md)
- [security-boundary.md](security-boundary.md)
- [security-boundary-schema.md](security-boundary-schema.md)
- [execution-schema.md](execution-schema.md)

## 桌面端分发要求

若 agent 面向桌面端产品，推荐把本地 MCP server 的安装与加载纳入 sandbox/execution 边界，而不是作为 ad-hoc 插件系统单独实现。

默认兼容目标是 MCP Bundles（`.mcpb`）：

- bundle 安装
- bundle 校验
- bundle 加载
- bundle 更新
- bundle 卸载

参考：

- <https://github.com/modelcontextprotocol/mcpb>

## 支持 remote resources

执行层不应假设被操作资源与 harness 同网段或同宿主机。

因此接口应避免隐式本地依赖，例如：

- 当前工作目录必须在本地文件系统
- 所有资源都能直接 syscall
- 所有工具都能直接读取本地环境变量

## 规范结论

- `sandbox` 模块必须显式区分 execution-level 与 environment-level 两层
- 当前仓库默认实现主要是 `Execution Sandbox`
- `Local Environment Sandbox` 应收窄为目录授权边界
- `Cloud Environment Sandbox` 应作为完整 cloud execution substrate 建模
- tool executor 与 sandbox 必须分离
- sandbox 规范必须允许 Local、Cloud 选择不同默认层级，而不改变接口语义
