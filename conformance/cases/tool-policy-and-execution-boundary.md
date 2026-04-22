# Case: Tool Policy And Execution Boundary

## 目标

验证 policy evaluation、tool execution 与 runtime tool loop 分层，而不是把 permission/policy、executor 与 turn orchestration 混成一层。

当前语义锚点：

- `tools/tool-model/policy-engine.md`
- `tools/tool-model/tool-executor.md`
- `harness/runtime/core/ralph-loop.md`

## Preconditions

- runtime 支持 tool policy evaluation 或语义等价 gate
- runtime 支持 delegated tool executor 或语义等价执行器
- tool invocation 结果可被 runtime tool loop 消费

## Ingress

1. 准备一个需要经过 policy gate 的 tool invocation
2. 触发 runtime tool loop 或语义等价执行路径
3. 观察 policy decision、executor invocation 与 runtime result handoff
4. 分别验证 allow 与 deny 两条路径中的至少一条

## Expected Runtime Semantics

- policy engine 负责判断是否允许，不负责真正执行 tool
- executor 负责执行 tool call，不负责重写 policy 决策
- runtime tool loop 负责消费 tool result 与 continuation，不等于 executor 本身
- deny、ask、allow 等 policy outcome 必须在 executor 之前生效，或以语义等价方式拦截执行

## Expected Persistent Effects

- policy decision 与 tool result 至少一者可被调用方外部观察
- deny path 不会伪装成成功执行后的普通 tool result

## Allowed Variance

- policy 与 executor 可以同进程，也可以跨边界调用
- executor 可以同步、异步或流式执行

## Failure Conditions

- executor 自行做最终 policy 决策而没有独立 policy 层
- runtime tool loop 与 executor 完全不可分离
- deny path 仍执行了 tool side effect
- policy outcome 无法被外部观察或追溯
