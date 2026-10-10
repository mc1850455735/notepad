
### 1. Session 级 Agent Runtime 编排

这是整个项目最有辨识度的部分。

核心内容：

- 每个活跃 Session 对应独立 Docker Runtime
- 挂载独立 Workspace、Sandbox、配置和凭证
- Runtime 内启动 Codex、Claude、Gemini、Grok 等官方 Server/CLI
- Bridge 统一不同模型的协议和流式事件
- Session 完成、中断、恢复、删除时管理容器生命周期

体现的能力：

- Docker 隔离
- 进程和资源生命周期管理
- Agent Runtime 架构
- 多模型接入
- 故障恢复与资源清理

### 2. `Session → Thread → Turn` 持久化会话模型

这是 AgentHub 区别于普通 Chat API 转发服务的关键。

核心内容：

- 一个 Session 绑定一个持久 Thread
- 每次用户补充产生新的 Turn
- Session 完成后仍可恢复原 Thread 继续对话
- 支持运行中追加消息、人工补充信息、中断与恢复
- PostgreSQL 保存控制面状态，Sandbox 保存运行产物

体现的能力：

- 状态机设计
- 多轮 Agent 生命周期理解
- 持久化和运行时状态一致性
- 异步任务编排
- 恢复执行设计

### 3. 多模型统一接入与协议适配

核心内容：

- OpenAI 使用 `codex app-server`
- Claude 使用官方 Server 或 CLI
- Gemini、Grok 使用官方 CLI
- 通过 Bridge 转换不同模型协议
- 动态获取模型和思考等级
- 处理模型能力差异、参数不兼容与回退
- 支持不同模型之间委派、投票和复核

体现的能力：

- Adapter/Bridge 模式
- 异构协议统一
- 流式事件处理
- 模型能力发现
- Agent 多模型调度

### 4. Agent 状态机与可靠性治理

核心内容：

```
Backlog → Todo → In Progress → Done
```

除此以外还涉及：

- success、failed、cancelled、timeout 等执行结果
- 并发执行保护
- 服务重启后的状态恢复
- 重复执行与重复消息处理
- Runtime 清理兜底
- Schedule 同一时刻只允许一个活跃实例
- 状态迁移事件记录

体现的能力：

- 状态机建模
- 幂等性
- 并发控制
- 异步任务可靠性

### 5. 多层存储与隔离设计

AgentHub 不是把所有内容都塞进数据库，而是分层存储：

|数据类型|存储位置|
|---|---|
|Session、Schedule、Turn、User|PostgreSQL|
|对话 transcript|Session Sandbox|
|用户附件|Sandbox uploads|
|Agent 产物|Workspace|
|Runtime HOME 与日志|Session Runtime 目录|
|长期记忆|Hindsight|
|Schedule 摘要记忆|`MEMORY.md`|

体现的能力：

- 控制面与数据面分离
- 多用户文件隔离
- 数据生命周期管理
- 数据库和文件系统协同
- 敏感环境变量和凭证隔离

### 6. Agent 记忆系统

该项目同时有两种记忆机制：

- `MEMORY.md`：Schedule 级、人类可读、有限大小的摘要
- Hindsight：Owner 级长期记忆，支持 recall 和 retain

执行链路包括：

```
SessionStart → 健康检查
UserPromptSubmit → recall
Stop / Done → retain
```

体现的能力：

- Agent 长期记忆
- 上下文注入
- 用户级数据隔离
- 记忆压缩和容量控制

### 7. 多入口接入与事件路由

AgentHub 不只有 Web，还包括：

- 飞书消息入口
- Jenkins / TeamCity 分析入口
- Service Token 内部 API
- Cron Schedule
- Web 即时会话

飞书 Bridge 还涉及：

- 消息幂等
- 回复消息到 Session 的映射
- 附件和图片下载
- 失败后的 fork / 新建会话回退
- 异步等待 Agent 最终结果
- 用户身份映射

体现的能力：

- 事件驱动系统
- 外部平台集成
- 消息路由
- 幂等处理
- 异步回调与失败降级

### 8. 渠道健康检查和凭证池治理

包括：

- 多个模型渠道配置
- OSS 托管 Codex 凭证池
- 周期性账号巡检
- `channel_probes` 健康记录
- 根据健康状态选择账号
- 渠道不可用时的告警和降级