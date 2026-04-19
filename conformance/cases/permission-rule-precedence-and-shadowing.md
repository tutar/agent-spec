# Case: Permission Rule Precedence And Shadowing

## 目标

验证 permission rule system 的优先级、遮蔽与 unreachable-rule 语义。

## Preconditions

- 存在至少一个支持 tool-wide rule 与 content-specific rule 的工具
- runtime 支持 `allow / deny / ask`
- 实现若支持 unreachable-rule detection，应能解释 shadowed rule

## Ingress

1. 配置一个 tool-wide `deny` 规则
2. 为同一工具再配置一个更细粒度 `allow` 规则
3. 重复上述过程，改为 tool-wide `ask` 与更细粒度 `allow`
4. 配置一个 content-specific `ask` 规则，并在 `bypass_permissions` 或语义等价模式下执行匹配动作

## Expected Runtime Semantics

- tool-wide `deny` 必须优先于更细粒度 `allow`
- tool-wide `ask` 必须优先于更细粒度 `allow`
- content-specific `ask` 不得被 bypass 模式静默绕过
- 若实现支持 shadow detection，被遮蔽的 `allow` 应可被解释为 unreachable

## Allowed Variance

- 是否对 unreachable-rule 提供单独 API 可实现自定
- shadow explanation 的文本格式可不同

## Failure Conditions

- `allow` 覆盖了 tool-wide `deny`
- `allow` 覆盖了 tool-wide `ask`
- bypass 模式绕过了 content-specific `ask`
- shadowed rule 仍被当作可达规则执行
