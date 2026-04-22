# Skills

## 职责

`Skills` 子模块负责定义 Agent Skills 的完整支撑面。

对 `AgentRuntime` 来说，这一组接口回答的是：

- startup / refresh / context assembly 阶段如何发现并导入 skills
- 当前 interaction 如何向模型或用户披露 skill catalog
- skill 激活后如何进入上下文并保持 compaction-safe continuity
- runtime 如何通过桥接层把 skill 暴露给模型调用

在当前规范中：

- `Skill` 属于 `tools` 域
- `Skill` 不是 `Tool`
- `Skill` 的默认运行时载体可以是 `Command`
- `Skill` 首先服务 Agent Skills 生态

## 阅读结构

- [discovery-and-import.md](discovery-and-import.md)
- [skill-definition.md](skill-definition.md)
- [skill-disclosure-and-activation.md](skill-disclosure-and-activation.md)
- [skill-context-management.md](skill-context-management.md)

## 规范结论

- `Skills` 规范必须覆盖 import、definition、disclosure、activation、context management 的完整生命周期
- `SkillTool` 只是 bridge，不是 `Skill` 本体
- runtime-facing activation 不能抹平 skill source、trust、scope 与 precedence
