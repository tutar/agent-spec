# Permission Rules

## 职责

本文定义 permission rule system 的稳定语义：

- rule source
- rule behavior
- rule language
- matching precedence
- update / persistence boundary
- dangerous-rule stripping
- shadowed / unreachable rule detection

rule system 是 permission runtime 的输入之一，但不是全部。

## 规则对象

推荐最小对象：

```text
PermissionRule
  - source
  - behavior: allow | deny | ask
  - rule_value
```

```text
PermissionRuleValue
  - tool_name
  - rule_content?
```

当 `rule_content` 为空时，表示 tool-wide rule。

## 规则来源

推荐最小来源集合：

- `user_settings`
- `project_settings`
- `local_settings`
- `flag_settings`
- `policy_settings`
- `cli_arg`
- `command`
- `session`

推荐区分两类：

- shared source
  例如 `project_settings`、`policy_settings`、`command`
- personal source
  例如 `user_settings`、`local_settings`、`cli_arg`、`session`

这个区分应影响：

- warning / explanation
- managed-only behavior
- shadowed-rule 提示

## 规则语言

规范允许至少这几类稳定表达：

- tool-wide rule
  - `Tool`
- content-specific rule
  - `Tool(content)`
- shell exact match
- shell prefix match
- shell wildcard match
- file read/edit pattern match
- agent-type scoped rule

推荐语义：

- tool-wide rule
  匹配整个工具
- content-specific rule
  匹配工具输入中的一部分语义内容
- shell prefix rule
  表达某一前缀下的命令族
- shell wildcard rule
  表达模式匹配，而不是简单 startsWith
- file pattern rule
  表达 read/edit 路径授权，而不是单纯字符串等值

兼容性建议：

- 实现可支持 legacy alias normalization
- 但 alias 兼容不应成为核心必选语义

## 优先级与遮蔽

稳定优先级：

- `deny > ask > allow`

稳定遮蔽规则：

- tool-wide deny 会遮蔽更细粒度 allow
- tool-wide ask 会遮蔽更细粒度 allow
- content-specific ask 不可被 bypass_permissions 绕过
- safety-check ask 不可被 bypass / auto fast-path 误绕过

推荐实现 unreachable-rule detection，至少识别：

- tool-wide deny 遮蔽 content allow
- tool-wide ask 遮蔽 content allow

推荐输出：

```text
UnreachableRule
  - rule
  - shadowed_by
  - shadow_type
  - reason
  - fix?
```

## 路径规则

对 file read / edit 类规则，推荐具备以下语义：

- read 与 edit 至少分为两类权限平面
- pattern 应相对 source root 或等价 root 解析
- pattern language 可为 gitignore-like 或语义等价表达
- deny rule 必须先于 allow rule 执行
- symlink / canonical path / traversal 安全检查不得被 rule matching 绕过

`additional_working_directories` 应被视为 `PermissionContext` 的一部分，而不是规则语言本体。

## 更新与持久化

推荐最小更新对象：

```text
PermissionUpdate
  - add_rules
  - replace_rules
  - remove_rules
  - set_mode
  - add_directories
  - remove_directories
```

更新边界要求：

- session update 与 persisted update 必须分开
- `policy_settings` 或等价 managed source 应视为只读
- permission UI 或 SDK 不得假定所有 source 都可写

允许 managed-only 模式：

- 仅受管规则生效
- 非受管“always allow”入口被隐藏、拒绝或忽略

## 危险规则与裁剪

规范允许宿主在进入高风险 mode 前，对过宽 allow 规则做 stripping 或降级。

典型对象包括：

- 任意 shell execution 的 tool-wide allow
- 任意代码执行前缀或通配 allow
- 任意 subagent spawn allow

要求：

- stripping 必须可解释
- 被裁剪结果应进入审计平面
- 安全裁剪不应被误表达为普通用户 deny

## 规范结论

- permission rules 是结构化对象，不是随意字符串列表
- source、behavior、matching、update 和 explanation 必须同时定义
- deny / ask / allow 的优先级必须稳定
- rule system 必须支持 personal / shared / managed 的来源差异
- path rule 与 shell rule 都应被视为一等规则族，而不是 UI 细节
