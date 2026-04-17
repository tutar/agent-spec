# Agent Orchestration Modes

## 目标

本文档把源码中已经存在的 agent 编排方式抽象成跨语言 agent 实现可复用的标准模式。

它不描述某个具体实现类，而是定义：

- agent 可以以哪些模式被编排
- 每种模式的生命周期语义是什么
- orchestration、task、agent loop、execution target 如何分层

## 为什么需要模式层

如果只写一个笼统的 `spawn_agent()` 接口，会把几种不同语义混在一起：

- 阻塞式 worker
- 后台任务
- 长存活 teammate
- 远端执行 agent

源码已经证明这几种形态是不同的，因此规范应显式建模。

## 四种标准模式

### 1. Synchronous Worker

同步 worker 是最基础的 subagent 模式。

语义：

- 由父 agent 发起
- 立即运行
- 父 agent 阻塞等待结果
- worker 完成后结果回流父链路

最小要求：

- 独立 `agent_id`
- 独立 transcript 分支
- 独立上下文和 tool scope
- 明确的 terminal state

适合：

- 检索
- 分析
- 验证
- 一次性专门化处理

### 2. Background Agent Task

后台 agent 是 worker 模式的异步化形式。

语义：

- agent 作为 task 运行
- 调用方不必阻塞等待结果
- agent 生命周期与 task lifecycle 绑定
- 结果通过 notification / output handle / polling 暴露

最小要求：

- `task_id`
- `agent_id`
- 可查询状态
- 可终止
- 可恢复输出
- 可通知完成/失败

适合：

- 长耗时分析
- 后台执行
- 与主线程并行的 agent 工作

### 3. Persistent Teammate

persistent teammate 不是一次性 worker，而是长期存活的 agent actor。

语义：

- agent 可长期存在
- agent 可进入 idle 状态并等待后续输入
- agent 拥有稳定 identity
- agent 可被再次寻址和继续交互

最小要求：

- 稳定 `agent_id`
- 可寻址 identity
- 消息注入接口
- idle / active / terminal 状态区分
- transcript 可持续追加

适合：

- 多 agent team
- lead/teammate 结构
- 持续协作型工作

### 4. Remote Agent Task

remote agent 表示 agent 执行发生在远端 execution target。

语义：

- 本地 orchestration 只负责发起和跟踪
- agent loop 不一定在本地 harness 内执行
- 本地持有 handle、状态同步和终止能力

最小要求：

- 本地 `task_id`
- 远端 `session_ref` 或等价句柄
- 状态同步
- 输出查询
- 远端终止/归档语义

适合：

- 远端环境运行
- VPC / cloud execution
- 本地控制、远端执行

## 推荐标准对象

```text
AgentSpawnRequest
  - mode: sync_worker | background_task | persistent_teammate | remote_task
  - agent_type
  - prompt
  - description
  - identity?
  - isolation?
  - tool_scope?
  - permission_mode?

AgentHandle
  - agent_id
  - mode
  - status
  - session_ref?
  - task_ref?
  - output_ref?
  - terminal_state?
```

其中：

- `mode`
  明确编排形态
- `identity`
  主要用于 teammate 场景
- `session_ref`
  用于 transcript / session 分支或远端 session 关联
- `task_ref`
  用于后台或远端任务

## 推荐最小接口

```text
Orchestrator
  - spawn_agent(request) -> agent_handle
  - send_input(agent_id, message)
  - background_agent(agent_id)
  - terminate_agent(agent_id)
  - await_agent(agent_id)
  - recover_agent(handle_or_session)
```

其中：

- `send_input`
  对 persistent teammate 是必要接口；对一次性 worker 可选
- `background_agent`
  允许从前台运行中的 agent 转后台
- `recover_agent`
  恢复 task / transcript / remote session 关联

## 与 TaskManager 的关系

- `TaskManager`
  负责后台对象的生命周期记录和状态转换
- `Agent Orchestration Modes`
  负责定义 agent 以哪种语义被运行

要求：

- 所有 `background_task` 和 `remote_task` 必须映射到 task
- `persistent_teammate` 推荐也映射到 task
- `sync_worker` 可不暴露给最终用户，但内部仍应有可跟踪 identity

## 与 Agent Loop 的关系

- orchestration mode 不等于 agent loop
- agent loop 负责单个 agent 的推理与工具回路
- orchestration mode 负责它如何被启动、托管、寻址和终止

因此：

- 切到 background 不应要求重写 agent loop
- 切到 remote 不应要求重写 orchestration 语义

## 与 Session 的关系

每个 agent 模式都应能明确回答：

- transcript 写到哪里
- 是否有独立 session 分支
- 如何恢复
- 父子关系如何记录

推荐约束：

- `sync_worker`
  使用 sidechain transcript
- `background_task`
  使用 sidechain transcript + task output
- `persistent_teammate`
  使用持续追加 transcript
- `remote_task`
  使用本地 task metadata + 远端 session ref

## 当前源码映射

当前仓库里，这四种模式已有比较完整的默认实现映射：

- `sync_worker`
- `background_task`
- `persistent_teammate`
- `remote_task`

## 规范结论

- orchestration 模块应显式定义标准 agent 编排模式
- 不同模式必须有不同的生命周期语义
- `spawn_agent()` 不能只抽象出一个模糊入口，而不定义模式差异
- 默认实现可以 task-first，但规范层必须支持同步、后台、持续协作和远端执行四类模式
- `AgentHandle`、`task_ref`、`terminal_state` 应与 [../../object-model.md](../../object-model.md) 中的 canonical objects 对齐
