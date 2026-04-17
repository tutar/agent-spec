# Skill Context Management

## 职责

本文件定义 skill 激活后的上下文保护、去重、allowlisting 和长期管理语义。

它解决的是：

- skill 激活后如何避免被 compaction 意外清掉
- skill 是否已经激活过
- skill 目录和资源如何进入 allowlist
- skill 是否能以 subagent / delegated mode 执行

## 稳定接口

```text
SkillContextManager
  - mark_activated(skill_name, activation_ref)
  - is_already_active(skill_name) -> boolean
  - protect_from_compaction(activation_ref)
  - list_bound_resources(skill_name) -> resources
```

## 必须支持的语义

- skill activation 结果应可被 dedupe
- 已激活 skill 应能被 context governance 识别
- skill root / bound resources 可进入 host allowlist，避免每次读取都重复审批
- resource listing 与 resource eager loading 必须分离

## 与 compaction 的关系

若 skill activation 已进入当前 interaction 的关键上下文：

- compaction 不应静默抹掉 skill 的核心语义
- 若 skill content 被抽取成 summary，也应保留 activation identity

## 与 subagent / delegation 的关系

某些宿主允许 skill 在 forked / subagent 环境中运行。

这属于扩展能力，不是 Agent Skills 导入的最低要求。

但若实现支持，应明确：

- activation 是在主线程还是子代理
- skill context 是否跨 agent 传播
- 资源 allowlist 是否随 activation scope 一起传播

## 当前仓库映射

当前本地默认实现的相关信号主要散落在：


以及与 context governance、permission allowlisting、forked execution 相关的运行时实现。

## 规范结论

- 技能导入规范不能只覆盖 loading，还必须覆盖 activation 之后的长期语义
- compaction protection、dedupe、allowlisting 是 skill support 的核心部分
