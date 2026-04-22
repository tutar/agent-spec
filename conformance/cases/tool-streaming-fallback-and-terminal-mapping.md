# Case: Tool Streaming Fallback And Terminal Mapping

## 目标

验证 streaming tool execution、fallback 与 abort reason 能映射到统一 terminal/error 语义，而不是散落成实现私有状态。

当前语义锚点：

- `tools/tool-model/tool-streaming-execution.md`
- `harness/runtime/core/failure-and-terminal-states.md`

## Preconditions

- host 支持 streaming tool execution 或语义等价的分段输出执行
- runtime 支持 fallback、abort 或 terminal/error mapping
- tool result 可以在 streaming 与 non-streaming 路径之间保持语义一致

## Ingress

1. 触发一个优先走 streaming 的 tool invocation
2. 观察正常 streaming 成功路径
3. 再触发一条会进入 fallback、user interrupt 或 sibling error 的路径
4. 观察 terminal/error 映射

## Expected Runtime Semantics

- streaming 与 non-streaming 执行路径都必须归约到统一 terminal/error 语义
- fallback 不得改变成功/失败/中断的外部结论
- abort reason 至少可映射到结构化 terminal-like outcome 或 error class
- runtime 调用方不需要理解底层 streaming transport 才能判断最终结果

## Expected Persistent Effects

- terminal/error 结果可被稳定观察或恢复
- fallback 发生后，最终结果不会丢失原本的 abort/terminal 语义

## Allowed Variance

- fallback 可以是自动切换，也可以是 host 决策后的语义等价重试
- abort reason 命名可以不同，只要能稳定映射到统一 terminal/error 语义

## Failure Conditions

- streaming path 与 fallback path 产出互相冲突的终态
- user interrupt、fallback failure、sibling error 无法区分
- terminal/error 语义只存在于 transport 细节，外部无法稳定观察
