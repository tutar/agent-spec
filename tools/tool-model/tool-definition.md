# Tool Definition

## 职责

`ToolDefinition` 描述一个可被模型调用、可被权限系统审查、可被执行器调度的能力单元。

tool 不是普通函数。它至少同时服务于 prompt、权限、执行、UI 和日志五个子系统。

## 标准接口

```text
ToolDefinition
  - name
  - aliases
  - description(input, describe_context)
  - input_schema
  - call(input, tool_use_context, permission_hook, parent_message, progress_handler)
  - is_enabled()
  - is_read_only(input)
  - is_concurrency_safe(input)
  - check_permissions(input, tool_use_context)
  - map_result(result, tool_use_id) -> ToolResult
```

## 推荐扩展字段

- `aliases`
- `searchHint`
- `maxResultSizeChars`
- `isDestructive`
- `interruptBehavior`
- `requiresUserInteraction`
- `shouldDefer`
- `alwaysLoad`
- `validateInput`
- `preparePermissionMatcher`
- `resultMediaType`
- `supportsResultPersistence`

## 必须解决的问题

- schema 验证
- prompt 中的工具描述生成
- 权限判定
- 并发安全判定
- 中断语义
- 大结果策略
- 与模型原生 tool calling 的语义对齐
- tool result / validation error / execution error 的边界

## 默认分类

- 只读工具
  例如 grep、read、list。
- 读写工具
  例如 edit、write、bash mutate。
- 交互工具
  例如 ask-user、permission prompt。
- 调度工具
  例如 agent spawn、task update。

## 设计要求

- 工具必须声明是否只读
- 工具必须声明是否支持并发
- 工具必须声明最大结果策略
- 工具必须自带权限检查入口
- 工具输入必须有机器可读 schema
- 若底层模型已原生支持 tool calling，SDK 应直接复用其调用协议，而不是再造一套模型侧协议
- 若底层模型不支持，SDK 应允许用 prompt 协议或 planner 协议模拟工具调用
- 输入校验失败应优先映射为 validation / protocol error，而不是伪装成业务执行失败
- `check_permissions()` 应返回策略判定或语义等价对象，而不是只返回 UI 文案
- 工具成功结果与工具失败结果都必须能映射到 canonical `ToolResult` / `SdkError`

## 输出语义

推荐最小输出约束：

```text
ToolResult
  - tool_name
  - success
  - tool_use_id
  - content
  - structured_content?
  - error?
  - persisted_ref?
```

其中：

- `persisted_ref`
  用于大结果外存化后保留稳定引用
- `error`
  应使用 canonical `SdkError` 或语义等价对象

失败至少应区分：

- input validation failure
- permission denied / requires action
- execution failure
- protocol / mapping failure

## 当前仓库映射

- 统一契约见 [Tool.ts](../../cc/Tool.ts)
- 工具注册表见 [tools.ts](../../cc/tools.ts)

## 规范结论

- SDK 不应接受“只有 execute 的工具”
- 工具定义必须带元数据
- 工具定义必须与 runtime/policy/executor 对齐
- 工具定义必须独立于具体语言的类型系统
- 工具定义必须能稳定映射到 canonical result / error / persistence 语义
