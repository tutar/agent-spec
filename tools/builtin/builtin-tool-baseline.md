# Builtin Tool Baseline

## 目标

本文件定义跨语言、跨宿主实现中推荐保留的 `built-in tool baseline`。

这里的重点不是“宿主里存在一些内置工具”，而是：

- 哪些工具构成 agent coding/runtime 的最小闭环
- 每个工具要解决什么问题
- 不同 host 可以怎样提供等价实现

## 设计原则

- 规范稳定的是工具语义，不是某个具体宿主里的实现方式
- `Local / Cloud` 可以使用不同 backend，但应保留相同外部行为
- baseline 工具应尽量覆盖代码代理最常见的闭环能力
- 不属于 baseline 的平台工具、实验工具、厂商特有工具不应混入这里

## 每类 baseline 工具都应补足的统一约束

无论具体工具类型如何，baseline 文档都应回答四件事：

- 最小语义契约
- policy sensitivity
- result persistence expectation
- context mutation expectation

推荐默认分类：

- `read_only`
  例如 `Read / Glob / Grep / WebSearch`
- `mutating`
  例如 `Write / Edit / Bash`
- `interactive`
  例如 `AskUserQuestion`
- `orchestration_bridge`
  例如 `Agent / Skill`

## 分层

### 1. Required Builtin Tools

这些工具应视为 coding-oriented agent SDK 的默认 baseline。

实现可以提供等价能力，但不应改变工具语义。

### 2. Recommended Builtin Tools

这些工具不是最小闭环必需，但高度推荐保留。

### 3. Host-Conditional Builtin Tools

这些工具是否存在取决于 host profile、平台能力或部署形态。

## Required Builtin Tools

### Read

职责：

- 读取单个文件内容
- 支持按路径读取文本或等价内容
- 为后续分析、修改、验证提供真实源材料

为什么必须：

- 没有 `Read`，agent 很难以稳定方式理解代码或配置
- 许多 skill、review、verification 流程默认依赖文件读取

等价实现要求：

- `Local`
  通常是本地工作区读取或本机代理读取
- `Cloud`
  可是 remote workspace read，但必须保留路径、错误、权限语义

规范补充：

- 默认属于 `read_only`
- 默认应支持并发安全执行
- 若结果超限，应允许通过 `PersistedToolResultRef` 暴露稳定引用

### Write

职责：

- 新建或整体写入文件
- 用于生成新文件、重写输出文件、写入持久化结果

为什么必须：

- 没有 `Write`，agent 无法稳定地产出新文件型结果

等价实现要求：

- 必须区分“创建/覆盖写入”和增量编辑
- 必须受 sandbox / policy 约束

规范补充：

- 默认属于 `mutating`
- 默认不应被视为并发安全
- 若执行阶段生成上下文修改，应通过 staged `ContextModifierCommit` 进入统一提交链路

### Edit

职责：

- 对已有文件做局部修改
- 保持最小 diff，而不是强制整体重写

为什么必须：

- 代码修改最常见的是局部 patch，不是整文件覆盖
- 没有 `Edit`，很多精细修复会退化成高风险重写

等价实现要求：

- 应支持对现有文件进行结构化或行级局部变更
- 应保留失败、冲突、定位不到目标等错误语义

规范补充：

- 默认属于 `mutating`
- 默认不应被视为并发安全
- 应与 conflict / not_found / validation 类错误区分明确

### Glob

职责：

- 基于模式列出匹配文件
- 提供目录结构探索与候选文件发现能力

为什么必须：

- 没有 `Glob`，agent 很难快速发现相关文件集合
- skill、plugin、memory、codebase exploration 常依赖它

等价实现要求：

- 可以是本地文件系统 glob，也可以是远端 workspace search
- 语义上应返回文件匹配结果，而不是全文搜索结果

规范补充：

- 默认属于 `read_only`
- 默认可并发

### Grep

职责：

- 在文件集合中搜索文本模式
- 支持按关键词或正则查找代码线索

为什么必须：

- 这是代码代理最核心的定位能力之一
- 无 `Grep` 时，很多代码定位会退化为低效遍历

等价实现要求：

- 语义上应是内容搜索，不是文件名搜索
- 可由本地 grep、索引搜索、远端搜索后端实现

规范补充：

- 默认属于 `read_only`
- 大结果应支持外存化引用，而不是无上限注入上下文

### Bash

职责：

- 执行 shell 命令或等价系统命令
- 支持测试、构建、检查、脚本运行、环境探测

为什么必须：

- coding agent 不只读写文件，还必须验证代码是否能运行
- verifier、build/test、repo inspection 基本都依赖这层

等价实现要求：

- `Local`
  通常是本地 shell / process execution，或本机代理 shell
- `Cloud`
  通常是 remote execution / environment sandbox shell

注意：

- 规范稳定的是“命令执行语义”，不要求一定叫 Bash，也不要求一定是 Unix shell
- Windows/PowerShell 或远端 execution proxy 也可以作为等价实现

规范补充：

- 默认属于 `mutating`
- 默认 policy sensitivity 最高
- 默认结果易超限，应优先支持 `PersistedToolResultRef`
- context mutation 与外部副作用语义必须与 `ToolExecutor` / `Sandbox` / `PolicyEngine` 对齐

### WebFetch

职责：

- 获取给定 URL 的页面或资源内容
- 用于读取文档、API 页面、公开网页内容

为什么必须：

- 很多 agent 任务需要直接读取指定网页，而不是搜索

等价实现要求：

- 应保留 URL 定向获取语义
- 应受网络 policy / sandbox / allowlist 控制

规范补充：

- 默认属于 `read_only`
- 结果可能超限，应支持 preview + stable ref 语义

### WebSearch

职责：

- 按查询词搜索外部网络信息
- 用于发现文档、新闻、网站、公开资料入口

为什么必须：

- 仅有 `WebFetch` 无法解决“先发现，再读取”的问题

等价实现要求：

- 可由 provider-native search、外部搜索 API、host-integrated search 提供
- 对上层应保持“搜索结果列表”语义，而不是直接网页正文

规范补充：

- 默认属于 `read_only`
- 不应伪装成 `WebFetch`

### Agent

职责：

- 创建子 agent、fork worker、后台 agent 或 teammate
- 把多 agent 编排暴露给模型或上层 runtime

为什么必须：

- 现代 agent runtime 很多非平凡任务都依赖 subagent/fork

等价实现要求：

- 至少应支持一种标准 agent spawn 语义
- 应与 orchestration/task lifecycle 对齐

规范补充：

- 默认属于 `orchestration_bridge`
- 它返回的不是普通文件结果，而是 agent/task linkage 或语义等价对象
- policy、interrupt、resume 语义必须和 orchestration 一致

### Skill

职责：

- 调用 prompt/workflow 型能力
- 把 skills 生态桥接到模型可调用平面

为什么必须：

- skills 是 Claude Code/Agent Skills 生态的重要入口
- 不保留 `Skill`，skill registry 很难无损进入 runtime

等价实现要求：

- `Skill` 是调用桥，不要求 skill 自身是 tool 对象
- 应支持本地 skills、plugin skills、MCP skills 等多来源能力

规范补充：

- 默认属于 `orchestration_bridge`
- 不应抹平 skill 原始来源
- 作为 bridge tool 时，应继续保留 command / skill surface 的原语义

### AskUserQuestion

职责：

- 在 agent 需要明确用户输入时发起结构化提问
- 为权限、澄清、分歧选择提供用户回路

为什么必须：

- 不是所有阻塞都能靠自动策略解决
- coding agent 仍需要一个标准化的人机澄清面

等价实现要求：

- 应与 gateway / interaction loop / requires_action 配合
- 不要求 UI 形式相同，但要保留“结构化问题 -> 用户答复”的语义

规范补充：

- 默认属于 `interactive`
- 不应被建模为 destructive mutating tool
- 应直接对齐 canonical `RequiresAction`

## Recommended Builtin Tools

### TodoWrite

- 用于显式维护任务拆解与执行清单
- 对较复杂任务和 plan mode 很有帮助

### TaskOutput

- 用于读取后台 task 的输出
- 对 background agent / shell task 很重要

### TaskStop

- 用于终止运行中的 task
- 对后台执行与长任务治理很有帮助

### NotebookEdit

- 用于编辑 notebook / 富结构文档
- 在数据科学、研究、教学场景下价值明显

## Host-Conditional Builtin Tools

### LSP

- 面向语言服务、符号、引用、诊断
- 对具备语言工具链的本地宿主很有价值

### EnterWorktree / ExitWorktree

- 面向工作区隔离
- 在多分支、多工作树和云端隔离环境中特别有价值

### PowerShell

- 适用于 Windows-first host
- 通常作为 `Bash` 语义的宿主特化，而不是额外 baseline

### Sleep / Monitor / Cron / RemoteTrigger

- 更偏自动化和长生命周期 agent
- 不应作为所有宿主默认 baseline

## 不应放入 Baseline 的工具

以下工具可以存在于具体产品中，但不应进入通用 baseline：

- 测试用工具
- 内部诊断工具
- 厂商私有工具
- 强 feature-gated 实验工具
- 只服务某个单独产品面板的工具

## 默认实现映射

当前仓库中的默认映射见 [../../tools.ts](../../cc/tools.ts)。

其中最接近 baseline 的内置工具包括：

- `Bash`
- `Read`
- `Write`
- `Edit`
- `Glob`
- `Grep`
- `WebFetch`
- `WebSearch`
- `Agent`
- `Skill`
- `AskUserQuestion`

当前仓库中还包含大量 recommended / host-conditional / internal tools，但它们不应被自动提升为跨实现 baseline。

## 规范结论

- `Builtin Tool Baseline` 应单独成文，不应只是 README 中一句话
- baseline 工具应明确枚举，并说明职责与等价实现要求
- 规范稳定的是工具语义，而不是具体宿主 backend
- `Local / Cloud` 可以实现不同，但应共享同一 baseline 行为
- baseline 工具文档必须同时回答 policy、persistence、context mutation 和 host variance 边界
