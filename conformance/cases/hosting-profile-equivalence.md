# Case: Hosting Profile Equivalence

## 目标

验证同一规范语义在 `Local / Cloud` 两种 host 下保持一致。

当前语义锚点：

- `module-overview.md`
- `orchestration/cloud/README.md`
- `harness/deployment-boundaries.md`

## Preconditions

- 两种 host profile 都实现了相同版本的 spec
- 两种 host 都支持同一个最小 turn、tool roundtrip、requires_action 场景

## Procedure

分别在：

- `Local`
- `Cloud`

下运行以下子场景：

- `basic-turn`
- `tool-call-roundtrip`
- `requires-action-approval`
- `session-resume`

## Expected Equivalence

两种 host 必须保持以下外部语义一致：

- session lifecycle 转移
- terminal state 分类
- `requires_action` 结构
- `tool_use -> tool_result` 配对规则
- resume 后的继续执行语义
- `Local-first` 默认实现不会改变 `Cloud-compatible` 边界语义

## Allowed Variance

允许不同的：

- 进程结构
- UI 交互形式
- 本地或远端持久化位置
- sandbox backend

不允许不同的：

- runtime event 语义
- session 边界语义
- tool / approval / continuation 语义

## Failure Conditions

- 某个 host 通过额外隐式状态实现恢复，导致外部行为与其他 host 不等价
- 某个 host 把 `requires_action` 降格为纯文本提示
- 某个 host 改变 tool roundtrip 的可观察语义
