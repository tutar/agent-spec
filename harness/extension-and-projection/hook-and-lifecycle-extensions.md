# Hook And Lifecycle Extensions

## 职责

`HookAndLifecycleExtensions` 负责在不改写核心 harness loop 的前提下，为 runtime 提供可插拔控制点。

它不是 tool 定义层，也不是 plugin 分发层，而是 lifecycle extension plane。

## 应覆盖的事件面

规范上至少应允许以下几类事件：

- session start / end
- turn stop / stop failure
- subagent start / stop
- teammate idle / task completed
- pre-tool / post-tool / post-tool-failure
- permission request / permission denied
- pre-compact / post-compact
- notification / config change

实现不要求事件名与当前源码完全一致，但必须保留等价语义。

## 稳定接口

推荐最小接口：

```text
HookRegistry
  - register_hook(scope, event, matcher, hook)
  - remove_hook(scope, hook_id)
  - list_hooks(scope, event?)
  - resolve_matching_hooks(event, input, scope) -> matched_hooks

HookRuntime
  - execute_hooks(event, input, runtime_context) -> hook_results
  - emit_hook_events(event_execution) -> external_events

HookResult
  - outcome: success | blocking | non_blocking_error | cancelled
  - message?
  - blocking_error?
  - system_message?
  - additional_context?
  - metadata
```

推荐补充：

```text
FunctionHook
  - id
  - event
  - matcher
  - callback(context) -> boolean | structured_result

FrontmatterHookRegistration
  - source_type
  - source_id
  - register_into_scope(scope)
```

## 设计要求

- hook 与 tool 必须分层
- hook 执行与 hook 结果投影必须分层
- hook 不应直接改写 session transcript，而应通过结构化结果影响 runtime
- hook 必须支持 session-scoped 注册
- hook 应支持进程内 function hook 与外部 command/prompt/agent hook
- hook 可影响 continuation、approval、blocking，但不得绕过 policy 与 session lifecycle 语义
- hook batch 可并行执行，但最终结果必须结构化聚合
- 当 hook 参与权限链时，必须有稳定优先级收敛规则
- hook 匹配必须支持 event-specific match key
- hook 匹配后应支持 source-aware dedup
- hook 应支持第二层条件过滤，例如 `if` 条件
- 实现应允许声明某些 hook 类型与某些事件不兼容

## 结果投影原则

hook 运行结果可以投影到多个平面，但必须保持语义分层：

- 对 message pipeline 的投影
  例如 progress、attachment、blocking feedback
- 对 policy chain 的投影
  例如 allow / deny / interrupt 建议
- 对 session lifecycle 的投影
  例如 stop continuation、requires_action
- 对独立 hook event stream 的投影
  例如 started / progress / response

禁止把这些投影混成一个字符串消息。

另外，lifecycle 扩展产出的 `additional_context` 不应被视为可以直接绕过 context assembly 的隐式输入。

推荐最小桥接对象：

```text
LifecycleContextEffect
  - lifecycle_scope: session_start | agent_start | turn | compact | stop
  - content
  - first_turn_only?
  - reentry_policy?
  - transcript_visibility
  - provenance
```

默认语义：

- session-start hook output -> session-start context
- agent-start hook output -> agent-start context
- 其他 lifecycle output -> 由对应上下文平面决定落点

实现必须保证：

- lifecycle 产物先进入 context assembly，再进入模型输入
- lifecycle 产物接受与普通上下文相同的 dedup、budget、governance、provenance 规则
- lifecycle 扩展不得通过旁路直接污染 bootstrap prompt 或 transcript

权限型 hook 结果推荐至少支持：

```text
PermissionHookEffect
  - behavior: allow | ask | deny | passthrough
  - updated_input?
  - reason?
```

推荐优先级：

- `deny`
- `ask`
- `allow`
- `passthrough`

## 与其它模块的边界

- 不等于 `Tools`
- 不等于 `PolicyEngine`
- 不等于 `SessionLifecycleState`
- 不等于 `Plugin`

它是：

- harness 的扩展面
- policy、session、tools 的旁路控制面

## 默认实现策略

推荐把 hook 执行分成三步：

1. 事件归一化
2. hook 匹配、去重、条件过滤
3. 结果聚合与投影

推荐允许：

- 并行执行多个匹配 hook
- 为单个 hook 设置独立 timeout
- progress / started / response 独立事件流
- internal callback hook 的轻量 fast-path

推荐至少支持三种 hook 类型：

- external command hook
- prompt / agent hook
- function hook

## 默认实现映射

当前仓库中的默认实现映射为：


## 规范结论

- hooks 应被建模为 lifecycle extension plane
- 任何需要在 turn、tool、approval、session 边界插入控制逻辑的 agent，都应保留 hook 接口
- hook 是运行时控制面，不是模型能力面
- hook runtime 应支持并行执行与多通道结果聚合
- hook runtime 应支持 event-specific matching model
