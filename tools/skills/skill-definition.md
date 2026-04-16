# Skill Definition

## 职责

本文件定义 Agent Skills 的目录结构、`SKILL.md` 解析规则和 manifest 兼容语义。

它解决的是：

- skill 文件长什么样
- 哪些 frontmatter 字段属于标准导入层
- 哪些字段属于 host-specific extension
- YAML / frontmatter 解析失败时如何宽松兼容

## 稳定接口

推荐最小对象：

```text
ImportedSkillManifest
  - name
  - description
  - license?
  - compatibility?
  - metadata?
  - allowed_tools?
  - skill_root
  - skill_file
  - source
  - diagnostics[]
```

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
```

## Agent Skills 标准兼容层

若实现声称支持 <https://agentskills.io/specification>，至少应兼容：

- 目录级 `SKILL.md`
- 标准 frontmatter 中的：
  - `name`
  - `description`
  - `license`
  - `compatibility`
  - `metadata`
  - `allowed-tools`（如宿主采用该实验字段）

推荐要求：

- `description` 缺失时可判为导入失败
- `name` 与目录名不一致时允许加载，但应记录 warning
- YAML 解析完全失败时允许 skip，但必须记录 diagnostics

## Host 扩展层

宿主可以支持额外字段，但必须和 Agent Skills 标准字段分层表达。

当前本地实现已体现的 host 扩展包括：

- `argument-hint`
- `arguments`
- `when_to_use`
- `version`
- `user-invocable`
- `model`
- `disable-model-invocation`
- `hooks`
- `context`
- `agent`
- `effort`
- `paths`
- `shell`

约束：

- 这些字段不应被伪装成 Agent Skills 基础字段
- 若进入共享规范，应作为 host-specific extension 或 `metadata` 子层表达

## 宽松解析规则

为兼容真实生态，导入实现应允许有限度的 lenient parsing：

- 第一次 YAML 解析失败后，可做一次 quoting / normalization retry
- cosmetic 问题可 warning + load
- 语义缺失或结构不可用时应 skip

推荐 diagnostics 分类：

- `warning`
  可继续导入
- `error`
  当前 skill 跳过

## 当前仓库映射

当前本地默认实现主要映射到：

- [../../../cc/skills/loadSkillsDir.ts](../../../cc/skills/loadSkillsDir.ts)
- [../../../cc/utils/frontmatterParser.ts](../../../cc/utils/frontmatterParser.ts)

源码中的关键信号包括：

- 支持宽松 YAML retry
- 支持 `allowed-tools`、`version`、`paths`、`shell` 等扩展
- 支持从 markdown body 中提取 description fallback

## 规范结论

- `Skills` 规范必须覆盖 `SKILL.md` 和 frontmatter 导入语义
- Agent Skills 标准层与 host 扩展层必须分层表达
- diagnostics 应成为导入协议的一部分，而不是实现细节
