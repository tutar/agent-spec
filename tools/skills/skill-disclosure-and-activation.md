# Skill Disclosure And Activation

## 职责

本文件定义 skills 如何向模型或用户披露，以及被激活后如何进入当前 interaction。

它回答的是：

- session start 时模型先看到什么
- 激活时返回什么
- skill content 何时注入
- skill resource 何时列出但不 eager load

## 披露层级

推荐采用 progressive disclosure：

### Tier 1. Catalog Disclosure

session 初始化时，只向模型披露：

- `name`
- `description`
- 可选 `location`

推荐对象：

```text
SkillCatalogEntry
  - name
  - description
  - location?
  - source
  - invocable_by_model
```

### Tier 2. Activation Disclosure

真正激活某个 skill 时，才注入完整 `SKILL.md` 内容或语义等价内容。

### Tier 3. Resource Disclosure

激活时可以列出：

- `scripts/`
- `references/`
- `assets/`

但不应默认 eager read 全部资源。

## 稳定接口

```text
SkillActivator
  - activate_skill(skill_id, args, context)
  - render_skill_prompt(skill_id, args, context)
```

推荐结果对象：

```text
SkillActivationResult
  - skill_name
  - body
  - frontmatter_mode: full | stripped
  - skill_root
  - listed_resources[]
  - wrapped: boolean
  - activation_mode: model | user
```

```text
SkillInvocationBridge
  - list_model_invocable_skills()
  - invoke_skill(skill_id, args, runtime_context)
```

## 激活模式

至少应支持：

- model-driven activation
  - 通过 bridge tool / activation tool
  - 或 host 明确支持的 file-read style activation
- user-explicit activation
  - `/skill-name`
  - `$skill-name`
  - mention / picker

## 包装与结构化返回

激活结果推荐使用稳定 wrapper 或结构化载荷，方便：

- compaction protection
- observability
- dedupe
- replay / restore

允许：

- full frontmatter
- stripped frontmatter + separate metadata channel

但宿主必须固定语义，不能在不同 turn 随机切换。

## 可见性与过滤

skill 在进入 catalog 或 activation tool 前，应先完成过滤：

- disabled
- host incompatible
- trust blocked
- user invisible
- model invocation disabled

约束：

- no-skills 情况下，不应输出空 catalog 噪音
- 不应让模型先看到一个必然不可激活的 skill

## 当前仓库映射

当前本地默认实现主要映射到：

- [../../../cc/skills/loadSkillsDir.ts](../../../cc/skills/loadSkillsDir.ts)
- [../../../cc/commands.ts](../../../cc/commands.ts)
- [../../../cc/tools/SkillTool/SkillTool.ts](../../../cc/tools/SkillTool/SkillTool.ts)

## 规范结论

- skill 支持面必须显式建模 disclosure 与 activation，而不只是 registry
- catalog disclosure、activation disclosure、resource disclosure 应三层分开
- `SkillInvocationBridge` 只负责桥接，不等于 skill 本体
