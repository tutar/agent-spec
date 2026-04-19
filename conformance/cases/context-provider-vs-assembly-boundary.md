# Case: Context Provider Vs Assembly Boundary

## 目标

验证 context provider 只负责产出结构化 fragment，而 assembly / governance 负责归并、排序和治理。

## Preconditions

- runtime 具有至少一个可观测的 context provider 概念或语义等价结构
- runtime 支持 attachment、structured context 或 dynamic recall 中至少两类 fragment source

## Ingress

1. 注册或调用多个不同来源的 context provider
2. 让这些 provider 分别返回不同 plane 的 fragment
3. 触发一次需要 attachment 排序或 budget 分析的普通 turn

## Expected Runtime Semantics

- provider 只产出 fragment，不直接决定最终 prompt 排序
- provider 至少应有等价于以下信息的语义：
  - plane
  - lifecycle
  - provenance / source identity
- assembly 层负责：
  - fragment 归并
  - attachment ordering
  - startup/per-turn 分层
- governance 层负责：
  - budget 分析
  - compact
  - externalization
  - delta planning

## Expected Runtime Effects

- 同一 provider 输出在不同 host profile 下可由本地或远程实现产生，但 fragment 语义一致
- 若 runtime 暴露调试或 observability 信息，能看出 provider 与 assembly/gov 层的职责分离

## Allowed Variance

- provider 可以是 direct-call、本地 store、远程接口、lazy fetch
- 不要求公开相同字段名，只要求职责边界稳定

## Failure Conditions

- provider 直接拼接最终 prompt
- provider 私自决定 attachment 最终顺序
- compact / budget / externalization 逻辑被埋在 provider 内且无法与 assembly/gov 分层
- 远程 provider 与本地 provider 输出的 fragment 语义不一致
