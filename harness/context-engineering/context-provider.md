# Context Provider

## 职责

`ContextProvider` 负责为某个 context plane 提供结构化上下文片段。

它不负责：

- 最终 prompt 拼接
- attachment 总排序
- token budget 决策
- compact / overflow recovery

这些职责属于 [context-assembly-pipeline.md](context-assembly-pipeline.md) 和 [context-governance.md](context-governance.md)。

## 标准接口

```text
ContextProvider
  - plane
  - lifecycle
  - audience
  - collect(runtime_state, assembly_request) -> ContextFragment[]
  - cacheability(fragment) -> Cacheability
  - provenance(fragment) -> Provenance
```

```text
ContextFragment
  - plane
  - payload
  - scope
  - provenance
  - cacheability
```

## 设计要求

### 1. provider 必须有明确 plane

provider 至少应明确自己服务于哪一类输入平面：

- `system_prompt`
- `system_context`
- `user_context`
- `attachment_stream`
- `capability_surface`
- `dynamic_context`

### 2. provider 必须声明 lifecycle

至少应区分：

- `startup`
- `per_turn`
- `resume`

否则 assembly 层无法判断何时重算、何时缓存、何时重放。

### 3. provider 必须保留 provenance

provider 输出不能是匿名字符串。

至少应支持：

- 来源标识
- scope
- cacheability
- 审计与去重所需的信息

### 4. provider 不拥有最终 ordering

provider 只负责提供 `ContextFragment`。

最终：

- system skeleton 的顺序
- attachment 的顺序
- dynamic recall 与 delta 的装配顺序

都必须由 assembly 层统一决定。

### 5. provider 可以本地，也可以远程

provider 可以是：

- direct-call 本地函数
- 本地 store 读取
- 远程接口返回的 context fragment

只要输出语义稳定，规范不关心部署形态。

## 默认策略

- bootstrap/system providers 提供稳定 system skeleton 或稳定 system context
- user/interaction providers 提供 user-side structured context
- attachment providers 提供 message-level 附加上下文
- recall providers 提供 memory / relevant evidence fragments
- capability providers 提供 tool surface 与 deferred/searchable capability 信息

## Local Mapping And Cloud-Compatible Mapping

### Local Mapping

- provider 常直接访问本地 session slice、rules files、tool registry、workspace snapshot

### Cloud-Compatible Mapping

- provider 可通过远程 session/tool/sandbox 接口取得结构化 fragment
- provider 依然只返回 fragment，不承担最终 assembly

## 与其它页面的边界

- [context-input-model.md](context-input-model.md)
  定义最终输入对象模型
- [context-assembly-pipeline.md](context-assembly-pipeline.md)
  定义这些 fragments 如何被装配
- [attachment-assembly.md](attachment-assembly.md)
  定义 attachment 的顺序、scope 与 audience
- [startup-and-turn-zero-context.md](startup-and-turn-zero-context.md)
  定义 startup-only context 的专门语义

## 规范结论

- `ContextProvider` 只负责提供结构化片段
- provider 必须声明 plane、lifecycle、cacheability 与 provenance
- provider 不拥有最终 ordering 和治理决策
