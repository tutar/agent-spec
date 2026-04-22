# Skills Discovery And Import

## 职责

本文件定义 Agent Skills 的发现、扫描、导入、去重和信任边界。

对 `AgentRuntime` 来说，这一页定义的是 startup / refresh / context assembly 前会调用的 skill import surface。

它回答的是：

- 从哪里发现 skills
- 哪些目录或包结构算作可导入 skill
- 导入时如何处理冲突、阴影覆盖和不可信来源
- `Local / Cloud` 下如何保持导入语义一致

## 稳定接口

推荐最小接口：

```text
SkillDiscoveryProvider
  - list_scopes(runtime_context) -> scopes
  - scan_scope(scope) -> discovered_skill_refs
  - resolve_precedence(skill_refs) -> deduplicated_skill_refs
```

推荐标准对象：

```text
DiscoveredSkillRef
  - skill_id
  - source
  - scope
  - skill_root
  - skill_file
  - trust_level?
  - diagnostics[]
```

## 默认发现位置

若宿主宣称兼容 Agent Skills 导入，推荐至少支持：

- project scope
  - `<project>/.agents/skills/`
  - `<project>/.<client>/skills/`
- user scope
  - `~/.agents/skills/`
  - `~/.<client>/skills/`

为兼容现有生态，宿主可额外支持：

- `.openagent/skills/`
- 祖先目录逐级发现
- bundled / managed / provisioned skill source

## 目录格式

默认只把“目录中存在 `SKILL.md`”视为标准 skill。

推荐结构：

```text
skill-name/
  SKILL.md
  scripts/
  references/
  assets/
```

其中：

- `SKILL.md`
  必需
- `scripts/`、`references/`、`assets/`
  可选

## 冲突与优先级

导入实现必须具备 deterministic precedence 规则。

推荐默认顺序：

1. project scope
2. managed / org scope
3. user scope
4. bundled scope

约束：

- 同名 skill 冲突时必须稳定地选择胜者
- 被覆盖的 skill 应产生 diagnostics，而不是静默消失
- 诊断信息必须可被 host、UI 或日志系统消费

## 信任与导入边界

project-level skills 不应在不可信仓库中默认静默启用。

宿主至少应能表达：

- trusted
- untrusted
- managed
- bundled

在不可信来源下，宿主可以：

- 禁止自动导入
- 只允许用户显式启用
- 降级为 catalog 可见但不可自动激活

## Cloud / Managed 导入

`Cloud-compatible` 实现不应假设所有 skills 都来自本地文件系统。

因此规范必须允许：

- project repo scan
- user / org provisioned skills
- bundled skills
- remote-distributed skill package

约束：

- 导入来源变化不能改变 skill 的外部语义
- path-based source 只是默认实现，不是规范前提

## 当前仓库映射

当前本地默认实现主要映射到：


源码中的关键信号包括：

- 支持 `skills / plugin / managed / bundled / mcp` 多来源
- 支持去重和同文件 shadowing 处理
- 支持 `.openagent/skills` 与 managed/user/project 范围

## 规范结论

- `Skills` 规范必须显式包含 discovery/import，而不只是 activation
- skill source、scope、trust、precedence 应成为一等语义
- Agent Skills 导入兼容性首先取决于 scan/import 语义是否稳定
