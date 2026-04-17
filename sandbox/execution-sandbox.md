# Execution Sandbox

## 目标

`Execution Sandbox` 规范单次工具执行级别的沙箱。

它主要面向：

- shell command
- code runner
- 单次文件操作或资源调用

它的典型实现不是长期运行环境，而是：

- 对一次执行请求做包裹
- 在执行期间施加文件系统、网络、进程等限制

## 适用层级

这一层对应：

- OS / 进程级沙箱
- 单次命令执行包装
- tool execution level isolation

它不要求一定是容器。

## 稳定接口

推荐最小接口：

```text
ExecutionSandbox
  - is_enabled() -> boolean
  - initialize(policy?) -> status
  - wrap_execution(execution_request, capability_profile) -> wrapped_request
  - execute(wrapped_request) -> execution_result
  - update_policy(policy)
  - reset()
```

建议补充：

```text
ExecutionSandboxViolation
  - violation_id
  - type: filesystem | network | process | environment
  - target
  - action
  - blocked
  - metadata?
```

## 设计要求

- `Execution Sandbox` 应是 per-execution 或 per-command 级别
- 应支持动态 policy 更新
- 应支持 violation reporting
- 不应要求 harness 本身运行在该沙箱内
- 应允许 ask callback 或 policy callback 处理网络/资源例外

## 默认实现映射

当前代码库的默认实现主要落在这一层：

  把 settings / permissions 投影成 `SandboxRuntimeConfig`
  决定某次 Bash 执行是否进入沙箱
  在 Bash 执行链中使用 sandbox wrapper

从源码可以看出，当前默认实现是：

- command wrapping
- process-level isolation
- filesystem/network restriction

而不是整个 agent 进程被放进容器。

## 要解决的问题

- 如何对单次执行施加 OS/进程级隔离
- 如何让工具执行使用统一的 sandbox policy
- 如何把 filesystem/network 违规作为标准事件暴露
- 如何支持 “进入沙箱” 与 “显式 unsandbox override” 的策略分支

## 规范结论

- 当前代码库的默认 sandbox 实现主要是 `Execution Sandbox`
- 这一层应被视为 `tool execution level sandbox`
- 它不等于容器，也不等于整个 runtime 的宿主环境
