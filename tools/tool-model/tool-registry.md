# Tool Registry

## 职责

`ToolRegistry` 负责把不同来源的执行型能力装配成当前 runtime 可见的 tool 集合。

它不是简单数组，也不是 UI 展示列表。它至少同时服务于：

- model-visible tool surface
- policy prefilter
- runtime resolution
- dynamic refresh
- capability provenance tracking

## 标准接口

```text
ToolRegistry
  - list_tools(scope?) -> tool_records
  - resolve_tool(name_or_alias) -> tool_definition?
  - filter_visible_tools(policy, runtime) -> tool_records
  - refresh(runtime_context?) -> tool_records
```

推荐标准对象：

```text
ToolRecord
  - tool
  - source
  - visibility
  - provenance?
  - feature_gate?
  - host_requirements?
```

其中 `source` 至少应区分：

- `builtin`
- `plugin`
- `mcp_adapter`
- `generated`

## 必须支持的语义

- tool resolution 必须稳定支持 name / alias
- visible tool filtering 与 execute-time policy 必须分层
- registry 必须保留来源信息，不能在 model surface 上完全抹平 provenance
- runtime refresh 如支持，必须保持解析结果与当前 host state 一致

## 与 Command / Skill / MCP 的边界

- `ToolRegistry`
  只承载执行型能力
- `CommandModel`
  承载 prompt / local / review 型入口对象
- `SkillRegistry`
  承载 skill 发现、加载、激活
- `McpProtocolClient`
  承载协议接入

某些能力可以桥接后同时出现在多个 surface，但原始来源语义必须保留。

## 当前仓库映射


## 规范结论

- `ToolRegistry` 应成为 `Tools` 模块的一等子规范
- registry 的职责是 capability assembly，不是权限执行器
- visibility filtering 与 execute-time authorization 必须严格分层
