# Skills

## 职责

`Skills` 子模块负责定义 Agent Skills 的完整支撑面。

它解决的不是“执行一个 tool”，而是：

- 发现与导入 skill
- 解析 `SKILL.md` 与 frontmatter
- 向模型或用户披露 skill catalog
- 在合适时机激活 skill
- 管理 skill 激活后的上下文、去重与 compaction 保护
- 通过桥接层把 skill 暴露给模型调用

在当前规范中：

- `Skill` 属于 `tools` 域
- `Skill` 不是 `Tool`
- `Skill` 的默认运行时载体可以是 `Command`
- `Skill` 首先服务 Agent Skills 生态

## 阅读结构

### 1. Discovery And Import

- [discovery-and-import.md](discovery-and-import.md)

这一页回答：

- skill 从哪里发现
- 如何扫描与去重
- 如何处理 trust、scope、precedence
- `Local / Cloud` 如何保持导入语义一致

### 2. Skill Definition

- [skill-definition.md](skill-definition.md)

这一页回答：

- `SKILL.md` 长什么样
- 哪些 frontmatter 字段属于 Agent Skills 标准层
- 哪些字段属于 host-specific extension
- 宽松解析与 diagnostics 如何表达

### 3. Disclosure And Activation

- [skill-disclosure-and-activation.md](skill-disclosure-and-activation.md)

这一页回答：

- 模型先看到什么 catalog
- skill 何时注入完整内容
- 资源何时列出但不 eager load
- skill 如何通过 bridge 被模型或用户激活

### 4. Context Management

- [skill-context-management.md](skill-context-management.md)

这一页回答：

- skill 激活后如何 dedupe
- 如何做 compaction protection
- 资源 allowlist 如何处理
- skill 是否以及如何在 subagent 中执行

## 稳定接口

推荐最小接口：

```text
SkillDefinition
  - id
  - name
  - description
  - content
  - arguments
  - when_to_use
  - allowed_tools
  - metadata

SkillRegistry
  - discover_skills(scope)
  - load_skill(skill_id)
  - invalidate_skills(scope)

SkillActivator
  - activate_skill(skill_id, args, context)
  - render_skill_prompt(skill_id, args, context)

SkillInvocationBridge
  - list_model_invocable_skills()
  - invoke_skill(skill_id, args, runtime_context)
```

推荐补充对象：

```text
ImportedSkillManifest
  - name
  - description
  - skill_root
  - skill_file
  - source
  - diagnostics[]

SkillCatalogEntry
  - name
  - description
  - source
  - invocable_by_model

SkillActivationResult
  - skill_name
  - body
  - skill_root
  - listed_resources[]
  - activation_mode: model | user
```

## 标准兼容层与扩展层

为支撑所有满足 <https://agentskills.io/specification> 的 skill 导入，
规范应至少区分三层：

- Agent Skills 标准兼容层
  - `SKILL.md`
  - 标准 frontmatter
  - 标准目录结构
- Pragmatic compatibility 层
  - 宽松 YAML 解析
  - 多 scope 扫描
  - 生态兼容路径
- Host-specific extension 层
  - model / effort / shell / hooks / fork context / path gating 等扩展字段

约束：

- host 扩展不能伪装成 Agent Skills 基础字段
- 扩展字段若需要进入共享规范，应通过 extension layer 或 `metadata` 明确表达

## 默认实现

当前代码库中的默认实现主要映射到：

- [skills/loadSkillsDir.ts](../../../cc/skills/loadSkillsDir.ts)
- [utils/frontmatterParser.ts](../../../cc/utils/frontmatterParser.ts)
- [types/command.ts](../../../cc/types/command.ts)
- [commands.ts](../../../cc/commands.ts)
- [tools/SkillTool/SkillTool.ts](../../../cc/tools/SkillTool/SkillTool.ts)

源码里的关键判断是：

- skill 的一等对象不是 `Tool`
- skill 默认由 `Command(type='prompt')` 承载
- `SkillTool` 只是桥接层
- 本地实现已支持多来源导入、宽松 frontmatter 解析和 host 扩展字段

## 当前源码映射

- `createSkillCommand()` 在 [skills/loadSkillsDir.ts](../../../cc/skills/loadSkillsDir.ts) 中把 skill 转成 `Command(type='prompt')`
- `loadedFrom` 用于区分 skill 来源，包含 `skills / plugin / managed / bundled / mcp`
- `parseFrontmatter()` 在 [utils/frontmatterParser.ts](../../../cc/utils/frontmatterParser.ts) 中提供宽松 YAML retry
- `getSkillToolCommands()` 在 [commands.ts](../../../cc/commands.ts) 中筛选模型可见 skill
- `SkillTool` 在 [tools/SkillTool/SkillTool.ts](../../../cc/tools/SkillTool/SkillTool.ts) 中调用 skill，并在必要时 fork sub-agent

因此，当前源码里的关系应理解为：

```text
Skill        -> 能力语义
Command      -> 默认对象载体
SkillTool    -> 模型调用桥接
```

## 规范结论

- `Skills` 规范必须覆盖 import、definition、disclosure、activation、context management 的完整生命周期
- `Skill` 是独立于 `Tool` 的稳定接口
- skill 的默认对象模型应允许 prompt/workflow 型表达
- skill 是否由模型调用，应由桥接层决定，而不是由 skill 定义本身承担
- `SkillTool` 不应被当成 `Skill` 本身
