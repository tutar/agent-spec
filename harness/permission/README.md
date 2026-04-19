# Permission

本目录收拢 `Harness` 域内与 permission runtime 直接相关的语义。

它回答几类问题：

- permission system 的对象模型、模式和运行时职责是什么
- permission rule 如何加载、匹配、更新和解释
- `ask -> requires_action` 如何投影、路由和恢复
- background / headless / worker 场景下如何做安全降级

这里的 permission 不是单一弹窗，也不只是工具侧的 `check_permissions()`。
它是一个跨 `Harness / Tools / Sandbox / Session` 的运行时子系统：

- `Harness`
  拥有审批投影、降级、路由、阻塞和恢复语义
- `Tools`
  提供 tool-specific permission check 与 matcher 接口
- `Sandbox`
  提供 working directory、filesystem 和 escalation 边界输入
- `Session`
  提供 `RequiresAction` 与 resume 的 durable 恢复语义

因此 permission 主规范放在 `Harness`，而不是单独落在 `Tools` 或 `Sandbox`。

## 目录内文档

- [permission-runtime.md](permission-runtime.md)
- [permission-rules.md](permission-rules.md)
- [approval-and-resume.md](approval-and-resume.md)
