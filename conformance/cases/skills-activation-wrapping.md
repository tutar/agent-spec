# Case: Skills Activation Wrapping

## 目标

验证 skill activation 的 disclosure 分层与结构化包装语义。

## Preconditions

- runtime 支持 skill catalog disclosure
- runtime 支持 skill activation

## Ingress

1. 暴露至少一个可激活 skill
2. 先读取 catalog disclosure
3. 再触发 activation

## Expected Runtime Semantics

- catalog disclosure 仅包含低成本摘要信息
- activation 返回完整或语义等价的 skill body
- activation 结果包含 skill root 或语义等价来源信息
- resource listing 与 resource eager loading 分离

## Failure Conditions

- catalog disclosure 泄露完整 skill body
- activation 结果没有稳定包装，无法 dedupe 或 compaction-protect
- resources 在 disclosure 阶段被无约束 eager load
