# Permission Runtime

## 职责

`PermissionRuntime` 负责把 execute-time authorization、结构化审批、无交互降级和恢复语义收敛成一套稳定系统。

它至少覆盖：

- permission mode
- permission context
- rule-based gate
- tool-specific permission check
- hook / classifier / automated checks
- working-directory / sandbox / safety-check 叠加裁决
- `ask -> requires_action` 投影
- background / headless / worker 场景下的安全降级

可见工具过滤与 execute-time permission 必须分层：

- visible tool filtering
  决定模型“看见什么”
- permission runtime
  决定运行时“能不能执行”

## 核心对象

推荐最小对象：

```text
PermissionMode
  - default
  - accept_edits
  - plan
  - bypass_permissions
  - dont_ask
  - auto
```

```text
PermissionContext
  - mode
  - additional_working_directories
  - always_allow_rules
  - always_deny_rules
  - always_ask_rules
  - should_avoid_permission_prompts
  - await_automated_checks_before_dialog
  - pre_plan_mode?
```

```text
PermissionDecisionReason
  - type: rule | mode | hook | classifier | working_dir | safety_check | async_agent | sandbox_override | other
  - details
```

```text
PermissionUpdate
  - add_rules
  - replace_rules
  - remove_rules
  - set_mode
  - add_directories
  - remove_directories
```

## 模式语义

- `default`
  普通 interactive approval 模式。
- `accept_edits`
  对工作目录内的安全 edit/write 提供快路径，但不覆盖 deny、显式 ask 或 safety check。
- `plan`
  收紧执行权限，并允许引入 session-scoped 临时规则。
- `bypass_permissions`
  绕过常规 prompting，但不得绕过 deny、content-specific ask 或 safety-check ask。
- `dont_ask`
  将最终 `ask` 降级成 `deny`。
- `auto`
  用 classifier / automated checks 取代人工审批；若无法自动裁决，则退化到 prompting 或 deny。

## 判定流水线

实现内部可以拆成多级函数，但外部语义应保持下列顺序稳定：

1. 静态 deny / ask rule
2. tool-specific permission check
3. bypass-immune safety checks
4. mode transform
5. allow rule
6. hook / classifier / automated checks
7. `ask -> requires_action` 或 `deny` 收敛

约束：

- deny 必须早于 allow
- tool-specific ask / deny 不能被后续快路径静默覆盖
- safety-check ask 必须早于 bypass / auto fast-path
- `passthrough` 只允许作为策略链内部语义，不得作为最终对外结果

## 交互与无交互语义

`PermissionRuntime` 必须显式区分三层：

- `PolicyDecision.ask`
  当前策略要求外部动作
- `RequiresAction`
  runtime 被结构化阻塞
- approval response
  恢复该阻塞的外部输入

若当前上下文不能直接触达用户：

- 先运行 hook / classifier / automated checks
- 若仍 unresolved，则必须：
  - auto-deny，或
  - 显式桥接到 leader / approval channel

禁止：

- 生成用户不可达的悬空 approval
- 把 unresolved ask 静默吞成 allow
- 把 `ask` 直接伪装成普通 tool failure

## 与 Sandbox 的边界

permission runtime 可以消费 sandbox 提供的输入：

- working-directory allowlist
- filesystem write allowlist
- sandbox escalation requirement
- dangerous path / safety-check 结果

但它不等于 sandbox。

稳定边界应为：

- `Sandbox`
  负责执行边界与 capability model
- `PermissionRuntime`
  负责 authorization、prompting、degradation、audit 和 resume binding

## 拒绝追踪

若实现包含 classifier、auto mode 或 automated denial，推荐引入 denial tracking：

```text
DenialTrackingState
  - consecutive_denials
  - total_denials
```

```text
DenialFallbackPolicy
  - record_denial(state) -> state
  - record_success(state) -> state
  - should_fallback_to_prompting(state) -> boolean
```

其职责是：

- 防止系统无限自动拒绝
- 在连续拒绝过多时退回 prompting / ask
- 在 headless 场景下安全收敛到 deny，而不是悬空阻塞

## 审计与可恢复性

permission 相关对象必须可序列化、可解释、可审计：

- `PolicyDecision`
- `PermissionDecisionReason`
- `RequiresAction`
- approval response
- `PermissionUpdate`

恢复要求：

- `ask` 必须绑定回原 tool action 或等价 action ref
- `deny` 不应形成 resumable pending approval
- restore 后必须能恢复最近一次 permission-triggered block 的上下文

## 规范结论

- permission runtime 属于 `Harness` 的稳定子域
- 它不是单一 UI dialog，也不是单纯 tool check
- permission mode、permission context 与 automated degradation 应视为一组标准对象
- safety-check、working-directory 和 sandbox override 都应进入同一 permission audit 平面
- background / headless / worker 场景必须显式建模，不得沿用前台审批的隐含假设
