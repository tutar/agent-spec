# Extension Packaging

## 职责

`ExtensionPackaging` 负责扩展包的声明、物化、安装、刷新和来源治理。

它不定义具体 tool/skill/agent 接口，而是管理这些能力如何被打包、分发并进入系统。

## 三层模型

推荐至少区分：

```text
ExtensionIntent
  - declared_sources
  - enabled_entries
  - config_origin

ExtensionMaterialization
  - fetch(source)
  - validate(package)
  - cache(version)
  - install(target)

ActiveExtensionComponents
  - load_components()
  - refresh_runtime_state()
  - swap_active_components()
```

## Builtin Extension Package

规范上应允许宿主自带一组官方扩展包。

这类扩展包的特点是：

- 随宿主发布，而不是由用户单独安装
- 共享 plugin / extension 的组件模型
- 可由宿主或用户启用/禁用
- 可在统一扩展管理界面中与 marketplace / user-installed extension 并列显示

推荐最小对象：

```text
BuiltinExtensionPackage
  - package_id
  - name
  - default_enabled
  - components[]
  - trust_level: builtin
```

需要注意：

- `builtin extension package` 是扩展分发层能力
- 它不同于 built-in command 或 built-in tool
- 它也不要求当前实现必须真的内置某些插件，只要求规范支持该形态

## 设计要求

- 扩展声明层与运行时激活层必须分开
- 版本化缓存必须是一等能力
- 来源校验与 policy 检查必须在物化层执行
- refresh 应建模为 active component swap，而不是简单重新扫描目录
- 扩展包必须允许承载多类能力，而不只是一种对象
- 应允许 `builtin extension package` 与 marketplace / session-only extension 共存

## 默认实现映射

当前仓库中的默认实现映射为：


## 规范结论

- plugin/extension packaging 是平台扩展层，不是 runtime 核心 loop
- 但对于需要生态分发的 agent，它应成为共享规范的一部分
- 宿主自带扩展包应被视为该层的可选能力，而不是硬编码特例
