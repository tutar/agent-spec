# Case: Task Output Cursor And Resume

## 目标

验证 task 的 `output_ref + cursor` 语义，以及 restore 后的稳定输出恢复。

## Preconditions

- task 支持稳定 output reference 或语义等价对象
- 调用方可按 cursor、offset 或等价机制增量读取 task 输出
- host 支持 restore / resume，或支持在任务运行后重建输出读取上下文

## Ingress

1. 启动一个会持续产出多段输出的 task
2. 读取第一段输出并记录当前 cursor
3. 再次读取后续输出，验证增量消费
4. 模拟 restore / resume
5. 用之前记录的 task identity 与输出句柄继续读取

## Expected Runtime Semantics

- task 输出通过稳定 `output_ref` 或等价句柄暴露
- 调用方可以从 cursor/offset 继续读取，而不是每次重新全量获取
- 新输出追加后，cursor 单调前进
- restore 后同一 task 仍能映射回同一输出语义
- restore 后至少可重建最近一次 task status、output handle 与最近 terminal-like stop reason

## Expected Persistent Effects

- 大输出不会要求全量内存驻留才能继续观察
- restore 后不会要求重新执行原 task 才能重新拿到同一输出引用
- 若 task 已终态，最后一次稳定结果或 terminal cause 仍可通过输出句柄或等价 durable 记录追溯

## Allowed Variance

- output transport 可以是 `file`、`stream`、`remote_log` 或 `transcript_branch`
- cursor 可以是字节 offset、逻辑序号或其他单调前进的读取位置
- restore 可以是显式 session 恢复，也可以是新的 observer 重新附着

## Failure Conditions

- task 输出只支持全量读取，不支持增量 cursor 语义
- restore 后 output reference 漂移到不同的执行结果
- 恢复输出必须重新执行原 task
- 新输出写入后无法从上一次读取位置继续消费
