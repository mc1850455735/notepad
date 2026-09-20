
# 项目中的 Agent Loop 是如何设计的

MewCoder 的 Agent Loop 本质上是一个基于 ReAct 思想的状态循环，负责串联：模型推理 -> 工具调用 -> 工具结果反馈 -> 模型继续推理 的整体流程。

当一个用户 Prompt 传入 Agent 后，首先会写入 Session 的 JSONL 事件日志，随后系统回放得到 SessionView，并由 ContextBuilder 构成一个完整的上下文信息，传入模型。

Agent 支持模型的流式响应和同步响应。在流式下，模型的普通文本会立即显示，但是 Tool Call 只有在流正常结束、收到 message_completed 且参数可以被完整解析为 JSON object 时才会被视为一个正常的 Tool Call。

拿到完整的模型响应后，Agent 会先持久化 Assistant Message。

对于没有 Tool Call 的情况，说明模型准备结束；
此时 Agent 还要检查 TaskPlan、后台任务和完成门禁，确认不存在未完成的任务或者未验证的修改后，真正结束本轮。

对于一个 Tool Call 的执行，主要分为以下几个步骤：
1. 检查工具是否已注册，对当前用户是否可见；MCP 工具还要检查是否已被检索激活
2. 对工具权限进行检查，得到 Allow、Ask、Deny 三种权限，并执行工具；
3. 工具执行完毕后，将 Tool Result 写入 JSONL；
4. 重新构建上下文，将 Tool Result 放入下次请求中，让模型进行下一步决策。

为避免 Loop 无限循环导致不必要的 Token 浪费，设置了一系列限制避免 Loop 无限运行：
- 设置最大工具轮次和 Provider 轮次，并设置 1 小时最高执行时间；
- 对超时、限流、网络和服务端错误进行有限轮次重试；
- 支持用户主动取消

暂停和恢复也被纳入 Loop。遇到 Ask 权限决策时，系统会保存可信 Tool Call 和权限请求，将 Loop 切换为 Pending 状态；用户确认后从原调用继续。
进程重启时，系统通过 JSONL 回放恢复上下文，并补齐中断的 Tool Call/Tool Result，保证 Provider 消息协议合法。

# MewCoder 中的上下文工程是怎么做的

MewCoder 的上下文管理采用
- 事件持久化
- 状态投影
- 动态组装
- 分级压缩
的架构，将完整会话事实和本轮模型输入分开管理。

整体流程是：
```
JSONL → SessionView → ContextManager → ContextBuilder → LLM
```

首先，所有会话事实都以事件形式**追加写**入 JSONL，包括用户消息、模型响应、Tool Call、Tool Result、TaskPlan、Checkpoint 和压缩记录。JSONL 保存完整历史，主要用于审计和恢复，不会直接全部发送给模型。

其次，系统回放 JSONL 生成 SessionView。SessionView 是当前有效状态的物化视图，会应用压缩替换、元数据更新和最新 TaskPlan，统一提供当前消息、Checkpoint 和任务状态。

每次调用模型前，ContextManager 会根据模型上下文窗口计算 Token 预算，当输入达到动态高水位时，系统按照 L1～L4 进行渐进压缩：

- L1：裁剪旧任务中的冗余普通文本。
- L2：按照代码、日志、搜索结果、Diff、JSON、HTML 等类型进行确定性压缩，保留错误、路径、行号和函数签名等关键信息。
- L3：将大型 Tool Result 完整写入基于 SHA-256 的内容寻址 Archive，上下文中只保留摘要和 `archive_id`，需要时通过 `retrieve_archive` 找回。
- L4：让模型把较早历史总结成结构化 Checkpoint，保留任务目标、关键决策、文件修改、测试结果和未完成事项。

每完成一层都会重新计算 Token，达到低水位后就停止，避免不必要的有损压缩。

最后，ContextBuilder 从最新 SessionView 中组装本轮模型真正需要的内容，其中包括：
+ 当前 TaskPlan
+ 最新 Checkpoint
+ Checkpoint 后的真实消息 Tail
+ 最新用户输入和必要 Tool Result

它还会校验 Assistant Tool Call 与 Tool Result 是否完整配对，避免压缩破坏 Provider 消息协议。

对于 Tool Result，系统会记录它是否进入过一次成功完成的模型请求。尚未被模型消费的结果不能进行有损压缩；当前用户需求、Pending Tool Call、当前任务的错误和测试证据也会优先保留。

所以，这套上下文管理并不是简单的固定滑动窗口，而是通过任务边界、信息生命周期和动态 Token 预算决定压缩优先级，再使用“Checkpoint + Recent Tail + Archive 按需召回”控制上下文规模。

# 项目中的长期记忆检索是怎么做的

当前项目还没有使用向量数据库或 BM25 自动检索跨 Session 记忆，也没有用户画像式的全局长期记忆。

MewCoder 当前的长期记忆采用：
- 会话回放
- Checkpoint 自动注入
- Archive 精确召回
三种方式。

第一种是**会话回放**。所有用户消息、模型响应、Tool Call、Tool Result、TaskPlan 和压缩事件都会追加写入 JSONL。恢复会话时，系统完整回放 JSONL，重建 SessionView 和 SessionRuntimeState。这部分可以被视为历史状态恢复。

第二种是 **Checkpoint 自动注入**。长对话经过 L4 压缩后，会生成结构化 Checkpoint，其中保存 `当前任务目标、已完成工作、关键技术决策、已修改文件、测试和错误信息、未完成事项及下一步` 等信息；
每次调用模型前，ContextBuilder 会自动找到最新 Checkpoint，并使用
`最新 Checkpoint + Checkpoint 后的真实消息 Tail` 的方法构建本轮上下文。

第三种是 **Archive 按需召回**。大型 Tool Result 会在 L3 阶段按照内容的 SHA-256 完整归档。在模型上下文中只保留内容摘要、`archive_id` 等信息；

如果模型需要查看原始内容，就调用 `retrieve_archive(archive_id, query, max_chars)` 方法，该方法用于定位归档，同时可以只返回关键信息的行并限制返回的数据量。该工具只允许读取当前 Session 的归档，并且读取时会校验 SHA-256，避免内容损坏或跨会话读取。

后续扩展跨会话长期记忆，我会在 ContextBuilder 之前增加 Memory Retrieval 层，先按用户、项目和权限进行元数据过滤，再结合 BM25 和向量检索召回 Top-K 记忆，经过重排后注入本轮上下文。

# 项目中的意图识别是怎么做的

项目中有轻量的意图识别，主要服务于上下文任务边界判断，而不是通用业务意图路由。针对上下文管理实现了 Task Boundary Detection，也就是判断用户的新 Query 是否切换了任务。

通过模型将新输入分类为：
- 同任务 Same
- 新任务 New
- 不确定 Uncertain

为了避免模型偶尔误判导致上下文频繁切换，系统不会看到一次 `new` 就立即切换，而是通过候选 Task Hash 和稳定窗口进行确认，默认连续稳定观察后才更新 `active_task_hash`。避免误判直接影响上下文压缩。

确认任务切换后，系统会：
1. 更新当前 Task Hash。
2. 将旧任务内容标记为更高优先级的压缩对象。
3. 重置当前任务相关的执行证据和停滞状态。
4. 触发相应的上下文治理流程。

# 项目中的 Unified Diff 以及文件快照二次校验识别过期审查是怎么做的？

这一步是一个 **写前预演 + 乐观并发校验** 的机制，目的是保证用户批准的内容和最终实际执行的内容一致。

Agent 使用工具对代码文件进行修改时，流程大致如下：
- 接受到 Tool Call
- 判断权限
- 根据文件内容生成修改
- 生成 Unified Diff 并保存文件快照
- 暂停 Agent 并等待用户确认
- 重新计算文件快照；一致时，执行原始的 Tool Call，不一致时，拒绝执行，并要求生成新的 Diff。

第一步是生成可信的修改预演。系统不会先修改文件再展示 Diff，而是根据 Tool Call 参数和当前文件内容在内存中计算修改前后的状态；

然后使用修改前后的文本生成 Unified Diff，并统计新增、删除行数和文件操作类型。生成 Diff 的同时，系统会为涉及的每个路径保存快照，快照主要是：文件路径 → 当前内容的 SHA-256。

系统只把 Diff 展示给用户，而原始 Tool Call、Permission Request 和 PrewriteReview 快照保存在服务端中。用户确认时只提交 Request ID 和选择，不能通过修改前端 Payload 替换工具参数。

用户批准后，系统不会立即执行，而是再次读取相关路径并计算摘要，与生成 Diff 时保存的快照进行比较，如果一致，说明用户审查之后文件没有变化，系统才执行修改。

如果任意路径摘要不同，说明审查期间文件可能被用户、IDE、其他 Agent 或后台进程修改。此时系统不会继续覆盖，而是返回结构化失败结果。该 Tool Result 会返回给模型，让模型重新读取文件、重新生成修改方案和新的 Diff。

需要说明的是，这种方式属于乐观并发控制，可以解决“生成 Diff 到用户确认”这段时间内的大部分过期审查问题，但它不是文件锁。快照校验和真正写入之间仍然存在很小的竞态窗口。如果进一步用于高并发企业环境，可以再增加文件锁、版本号或基于摘要的原子 Compare-And-Swap。


**例如**
Agent 准备把 `app.py` 中的 A 修改为 B，用户看到 Diff 后，另一个进程把文件修改成了 C。如果不进行二次校验，Agent 仍然按照旧内容写入，就可能覆盖 C；通过快照比较，系统会发现摘要已经变化并阻止这次写入。


# 上下文管理

## MewCoder 为什么需要上下文管理？如果直接把全部历史对话和 Tool Result 都发送给模型，会产生哪些问题？

MewCoder 需要上下文管理，主要是因为模型的上下文窗口有限，而 Coding Agent 会产生大量代码、日志、Tool Call 和 Tool Result。如果全部发送给模型，会带来四个问题：

1. 容易超过模型上下文窗口，导致请求失败。
2. Token 消耗和调用成本持续增加。
3. 输入内容过多会增加响应延迟。
4. 旧任务、重复结果和冗余日志会稀释当前需求，影响模型判断。

因此，项目将完整会话事实和模型本轮上下文分开管理：完整历史通过 JSONL 持久化，调用模型时再根据当前任务、Token 预算和信息生命周期选择必要内容，并通过渐进压缩和归档控制上下文规模。

## ContextManager 和 ContextBuilder 有什么区别？为什么不能把压缩逻辑直接写在 ContextBuilder 里面？

ContextManager 负责上下文治理。它会计算 Provider 输入的 Token 预算，当输入达到动态高水位时，执行 L1～L4 压缩，并记录压缩或 Checkpoint 事件。

ContextBuilder 负责上下文投影。它在每次模型请求前，从最新 SessionView 中组装 System Prompt、TaskPlan、Checkpoint 和最近消息，同时校验 Tool Call 与 Tool Result 的配对关系。

二者分开的原因不仅是触发时机不同，还因为 ContextManager 允许产生压缩副作用，而 ContextBuilder 应保持只读和确定性。这样上下文组装更容易测试，也能避免一次普通模型请求意外修改会话历史。

## L3 Archive 和 L4 Checkpoint 都能减少上下文长度，它们有什么区别？分别适合处理什么内容？

L3 Archive 主要处理单个大型 Tool Result。系统先按照内容的 SHA-256 保存完整原文，再在上下文中替换成包含摘要和 `archive_id` 的 Placeholder。模型需要查看原文时，可以调用 `retrieve_archive` 按需召回。因此，L3 属于可恢复的无损迁移。

L4 Checkpoint 主要处理较早的整段会话历史。系统让模型将任务目标、关键决策、文件修改、测试结果和未完成事项总结为结构化 Checkpoint，后续只发送 Checkpoint 和最近的真实消息 Tail。因此，L4 压缩范围更大，但属于有损语义压缩。

# 权限隔离

## `Allow`、`Ask`、`Deny` 分别代表什么？一次 Tool Call 进入系统后，权限模块会根据哪些信息作出决策？

权限决策结果分为 `Allow`、`Ask` 和 `Deny`。Allow 表示可以继续执行，Ask 表示需要用户确认，Deny 表示策略明确禁止执行。

完整 Tool Call 进入权限预检后，系统会根据工具名称和参数生成规范化的 Permission Request，其中包含权限动作和目标，例如写入的文件路径、执行的 Shell 命令或调用的 MCP 工具。PermissionManager 再结合默认策略、当前权限模式和已有授权记录作出决策。

Allow 会进入后续执行流程，文件修改还可能需要 Unified Diff 审查；Ask 会暂停当前 Agent Loop，并保存可信的原始 Tool Call 和权限请求；Deny 或用户拒绝时不会执行工具，但会生成对应的失败 Tool Result，保证 Tool Call 与 Tool Result 的 Provider 协议配对。


