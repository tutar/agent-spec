# Case: Skills Discovery Precedence

## 目标

验证 Agent Skills 导入时的 scope 扫描、冲突处理与 precedence 语义。

## Preconditions

- 至少两个 scope 中存在同名 skill
- runtime 支持 skill discovery / import

## Ingress

1. 在 project scope 提供一个 skill
2. 在 user 或 managed scope 提供同名 skill
3. 执行 discovery 与 dedup

## Expected Runtime Semantics

- precedence 结果 deterministic
- 被 shadow 的 skill 产生 diagnostics 或语义等价 warning
- 最终 catalog 中只暴露胜出的 skill

## Failure Conditions

- 同名 skill 选择结果不稳定
- shadowing 完全静默，无法诊断
- catalog 同时暴露多个冲突 skill
