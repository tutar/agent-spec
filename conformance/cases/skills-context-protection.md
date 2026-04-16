# Case: Skills Context Protection

## 目标

验证已激活 skill 在 compaction、重复激活和上下文治理下的保护语义。

## Preconditions

- runtime 支持 skill activation
- runtime 支持 context governance / compaction

## Ingress

1. 激活一个 skill
2. 再次触发同一 skill 或等价 activation
3. 触发 compact 或等价 context governance

## Expected Runtime Semantics

- skill activation 可以被 dedupe
- compaction 不会静默丢失 skill 的核心语义
- skill bound resources 的 allowlist 或语义等价机制保持有效

## Failure Conditions

- 重复激活导致无限重复注入 skill body
- compact 后 skill activation identity 丢失
- 激活后 skill 目录访问仍每次重复走冷启动审批
