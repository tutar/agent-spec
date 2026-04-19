# Case: Permission Mode And Headless Degradation

## 目标

验证 permission mode 的核心行为，以及 headless / background 场景下的安全降级。

## Preconditions

- runtime 支持至少 `default`、`dont_ask`、`bypass_permissions`
- 若宣称支持高级模式，还应支持 `accept_edits`、`plan`、`auto`
- 存在一个在默认模式下会产生 `ask` 的工具调用

## Ingress

1. 在 `default` 模式下执行一次会触发 `ask` 的动作
2. 切换到 `dont_ask`，重复相同动作
3. 切换到 `bypass_permissions`，执行一个仅由普通 prompting 阻塞、但不受 deny / safety-check 保护的动作
4. 在 headless / background / worker 上下文下执行一次会触发 `ask` 的动作
5. 若实现支持 `auto`，执行一次需要 classifier 或 automated check 的动作

## Expected Runtime Semantics

- `default`
  产生 `PolicyDecision.ask` 并投影为 `RequiresAction`
- `dont_ask`
  将最终 `ask` 降级为 `deny`
- `bypass_permissions`
  只能绕过可绕过的 prompting，不得绕过 deny、content-specific ask 或 safety-check ask
- headless / background
  不得生成用户不可达 approval；应 auto-deny 或桥接到显式 approval channel
- `auto`
  应先使用 classifier / automated checks，再退化到 prompt 或 deny

## Allowed Variance

- `accept_edits`、`plan`、`auto` 的具体触发条件可不同
- automated check 的底层技术可不同

## Failure Conditions

- `dont_ask` 仍然生成 `RequiresAction`
- `bypass_permissions` 绕过了 deny 或 safety-check ask
- headless 场景留下不可达 approval
- `auto` 失败时没有退化路径，或错误地直接 allow
