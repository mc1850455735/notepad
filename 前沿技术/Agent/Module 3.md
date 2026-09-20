# 流式输出

在 Python < 3.11 时，需要通过向 Node 中传递 RunnableConfig 配置来启用逐 Token 输出。
```python
# 模型调用节点, 向其中传入 RunnableConfig 启用逐 Token 输出
def call_model(state: State, config: RunnableConfig):
    
    # 如果存在 summary, 则获取
    summary = state.get("summary", "")

    # summary 存在, 则使用 summary
    if summary:
        # Add summary to system message
        system_message = f"Summary of conversation earlier: {summary}"
        # Append summary to any newer messages
        messages = [SystemMessage(content=system_message)] + state["messages"]
    else:
        messages = state["messages"]
    
    response = model.invoke(messages, config)
    return {"messages": response}
```

## 流式输出完整状态

对于一个 Graph 类型对象，`stream` 和 `astream` 分别是用于流式返回结果的同步方法和异步方法。

在 LangGraph 中，支持几种不同的流式输出模式。
- values：在每个节点被调用后，流式输出图的完整状态；
- updates：在每个节点被调用后，流式输出图状态的更新内容。

**updates**
当使用 updates 方式进行流式处理时, stream 传回的对象为每个节点响应组成的 chunk 列表，每个 chunk 都是一个字典，其中 node_name 作为键, 更新后的状态作为值.  每个节点执行完后，只返回这个节点产生的更新内容

```python
# Create a thread
config = {"configurable": {"thread_id": "1"}}

# Start conversation
for chunk in graph.stream({"messages": [HumanMessage(content="hi! I'm Lance")]}, config, stream_mode="updates"):
    print(chunk)
```

**values**
当使用 values 模式进行流式处理时, stream 返回值为当前完整的 graph state。

```python
# Start conversation, again
config = {"configurable": {"thread_id": "2"}}

# Start conversation
input_message = HumanMessage(content="hi! I'm Lance")
for event in graph.stream({"messages": [input_message]}, config, stream_mode="values"):
    for m in event['messages']:
        m.pretty_print()
    print("---"*25)
```



## 流式输出 token

在进行流式输出时，我们希望输出的不只是图状态，而是 LLM 逐步生成的 token 流。

为了实现这一点，可以使用 `astream_events` 方法，该方法会在节点内部事件发生时将事件流式返回。

流式返回的每个事件都是一个字典，其中包含几个键：
- `event`：正在发出的事件类型
- `name`：事件的名称
- `data`：与该事件相关的数据
- `metadata`：包含一系列信息，其中包含 `langgraph_node`，即发出该事件的节点名

```python
config = {"configurable": {"thread_id": "3"}}
input_message = HumanMessage(content="Tell me about the 49ers NFL team")
async for event in graph.astream_events({"messages": [input_message]}, config, version="v2"):
    print(f"Node: {event['metadata'].get('langgraph_node','')}. Type: {event['event']}. Name: {event['name']}")
```

对于 LLM 产生的 token，其 event 事件类型为 `on_chat_model_stream` 类型。

通过 event 属性，可以控制希望获取的事件类型；通过 metadata，可以控制希望看到的节点。通过控制这两个属性，可以使用 data 属性输出特定节点包含的特定类型的消息。

```python
async for event in graph.astream_events({"messages": [input_message]}, config, version="v2"):
    # Get chat model tokens from a particular node 
    if event["event"] == "on_chat_model_stream" and event['metadata'].get('langgraph_node','') == node_to_stream:
        print(event["data"])
```

只需使用 `chunk` 键，即可获取 `AIMessageChunk`，即返回消息的主要部分。

```python
async for event in graph.astream_events({"messages": [input_message]}, config, version="v2"):
    # Get chat model tokens from a particular node 
    if event["event"] == "on_chat_model_stream" and event['metadata'].get('langgraph_node','') == node_to_stream:
        data = event["data"]
        print(data["chunk"].content, end="|")
```

## LangGraph API 的流式输出

当把自己的 graph 部署到 studio 上时，可以通过 `stream` 方法进行流式输出，当使用 `values` 模式时，类似之前，每次流式返回都将返回完整的消息。

流式输出的返回值对象包括：
- `event`：流式输出的类型
- `data`：状态数据信息

```python
async for event in client.runs.stream(thread["thread_id"], 
                                      assistant_id="agent", 
                                      input={"messages": [input_message]}, 
                                      stream_mode="values"):
    print(event)
```

```
StreamPart(
	event='values', 
	data={
		'messages': [
			{
				'content': 'Multiply 2 and 3', 
				'additional_kwargs': {}, 
				'response_metadata': {}, 
				'type': 'human', 
				'name': None, 
				'id': '679c68ed-8f1f-4419-a774-b7b4aff01169'
			}
		]
	}, 
	id=None
)
```

同时, 存在一些只在 API 中支持的一些新的流式输出模式.
如: 可以通过 messages 模式更好的处理需要流失输出消息的情况.

其中, messages 模式下返回的所有消息都具有两个属性, 分别是:
- `event`：事件名称
- `data`：与该事件相关的数据

对于 event 事件名称，存在多种事件类型， 如：
- `metadata`：关于本次运行的元数据
- `message/complete`：完整形成的消息
- `message/partial`：聊天模型生成的 token

```python
thread = await client.threads.create()
input_message = HumanMessage(content="Multiply 2 and 3")
async for event in client.runs.stream(thread["thread_id"], 
                                      assistant_id="agent", 
                                      input={"messages": [input_message]}, 
                                      stream_mode="messages"):
    print(event.event)
```

# 断点

流式输出为 `Human-in-loop` 模式提供了基础，而 `Human-in-loop` 存在多种目的：

- 审批：允许中断 agent 并将状态展示给用户，并询问用户是否接受某个动作；
- 调试：可以允许用户回退图，以复现或避免某个问题
- 编辑：允许用户修改状态

LangGraph 提供了多种更新 agent 状态的方式，其中一种即为 `断点`

## 用于人工审批的端点

如果希望每次使用工具前，都由人工批准，只需要在编译图时设置：
`interrupt_before=["tools"]`、这意味着执行会在 `tools` 之前被中断。

```python
# Graph
builder = StateGraph(MessagesState)

# Define nodes: these do the work
builder.add_node("assistant", assistant)
builder.add_node("tools", ToolNode(tools))

# Define edges: these determine the control flow
builder.add_edge(START, "assistant")
builder.add_conditional_edges(
    "assistant",
    tools_condition,
)
builder.add_edge("tools", "assistant")

memory = MemorySaver()
graph = builder.compile(interrupt_before=["tools"], checkpointer=memory)

# Show
display(Image(graph.get_graph(xray=True).draw_mermaid_png()))
```

通过 `graph.get_state(thread)` 可以获取图的执行状态，并通过 `state.next` 可以获得下一个要执行的节点。

当向 `stream` / `invoke` 等图调用节点中传入 None 作为消息时，LangGraph 会从上一次状态 checkpoint 继续执行。

```python
for event in graph.stream(None, thread, stream_mode="values"):
    event['messages'][-1].pretty_print()
```

在本案例中，graph 会从上次被中断的 tools 节点继续执行。也就是说，传入 None 消息可以作为允许节点执行的方法使用。

一个结合 None 消息实现用户审批的流程如下：

```python
# Input
initial_input = {"messages": HumanMessage(content="Multiply 2 and 3")}

# 执行图并以 values 模式流式返回每一步骤的完整状态
# 直到达到断点
for event in graph.stream(initial_input, thread, stream_mode="values"):
    event['messages'][-1].pretty_print()

# 询问用户是否同意
user_approval = input("Do you want to call the tool? (yes/no): ")

if user_approval.lower() == "yes":
    # If approved, continue the graph execution
    for event in graph.stream(None, thread, stream_mode="values"):
        event['messages'][-1].pretty_print()
else:
    print("Operation cancelled by user.")
```

## 在 LangGraph API 中设置断点

当希望图可以实现断点功能时，可以在编译图时添加 `interrupt_before=["node"]` 参数，或者是在使用 API 时，直接将 `interrupt_before=["node"]` 参数传递给用于调用 API 的 `stream` 方法。

```python
initial_input = {"messages": HumanMessage(content="Multiply 2 and 3")}
thread = await client.threads.create()
# 直接在 stream 方法中指定 interrupt_before 的发生位置
async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id="agent",
    input=initial_input,
    stream_mode="values",
    interrupt_before=["tools"],
):
    print(f"Receiving new event of type: {chunk.event}...")
    messages = chunk.data.get('messages', [])
    if messages:
        print(messages[-1])
    print("-" * 50)
```

可以在 LangSmith 中观测到这个过程。当中断发生时，可以在 LangSmith 中手动允许执行，或者类似之前的方法，向 Graph 中传入一个为 `None` 的输入即可表示允许执行。

```python
async for chunk in client.runs.stream(
    thread["thread_id"],
    "agent",
    input=None,
    stream_mode="values",
    interrupt_before=["tools"],
):
    print(f"Receiving new event of type: {chunk.event}...")
    messages = chunk.data.get('messages', [])
    if messages:
        print(messages[-1])
    print("-" * 50)
```


# 编辑图状态

通过设置断点，我们可以在执行某个节点前对图的流转进行中断，并在下一个节点之前等待用户的审批。

同样的，借助断点，还可以在图被中断后修改图的状态。当图被中断后，使用 `add_messages reducer` 方法对 `messages` 键进行更新。使用如下：
* 如果我们想覆盖已有消息，可以提供该消息的 `id`。
* 如果我们只是想把消息追加到消息列表中，那么可以传入一条未指定 `id` 的消息

## 追加消息列表

当不传入指定 id 时，会对消息列表进行追加

```python
# 不指定 id, 即在消息列表中加入一条新消息
graph.update_state(
    thread,
    {"messages": [HumanMessage(content="No, actually multiply 3 and 3!")]},
)
# 展示修改后的消息列表
new_state = graph.get_state(thread).values
for m in new_state['messages']:
    m.pretty_print()
```

对消息列表进行修改后，传入 None 允许图从当前状态继续向下执行。
```python
for event in graph.stream(None, thread, stream_mode="values"):

    event['messages'][-1].pretty_print()
```

## 编辑图状态

LangGraph API 支持编辑图状态。

就像之前说的，使用 API 时，可以不在编译图使设置断点，而是在调用时主动传入断点的位置，如下：
```python
initial_input = {"messages": "Multiply 2 and 3"}
thread = await client.threads.create()
async for chunk in client.runs.stream(
    thread["thread_id"],
    "agent",
    input=initial_input,
    stream_mode="values",
    interrupt_before=["assistant"],
):
    print(f"Receiving new event of type: {chunk.event}...")
    messages = chunk.data.get('messages', [])
    if messages:
        print(messages[-1])
    print("-" * 50)
```

同样的，可以通过 `get_state` 方法访问当前的 state 状态，并获取消息的 id 以实现修改消息等目的。
```python
current_state = await client.threads.get_state(thread['thread_id'])
last_message = current_state['values']['messages'][-1]
last_message['content'] = "No, actually multiply 3 and 3!"
```

最后，将经过修改的、带有 id 信息的 `last_message` 传入 `update_state` 方法中。
```python
await client.threads.update_state(thread['thread_id'], {"messages": last_message})
```

同理，通过传入 None 恢复图的执行。
```python
async for chunk in client.runs.stream(
    thread["thread_id"],
    assistant_id="agent",
    input=None,
    stream_mode="values",
    interrupt_before=["assistant"],
):
    ...
```

## 获取用户输入

当 agent 进入断点后，可以对 agent 的状态进行编辑，而如果想要允许人工反馈执行状态更新，可以添加一个占位节点 `human_feedback` 用于人工反馈。

对于占位节点  `human_feedback`，使用 `interrupt_before` 属性在 `human_feedback` 节点之前设置断点，并通过 checkpoint 保存图执行到该节点之前的状态。

```python
# no-op node that should be interrupted on
def human_feedback(state: MessagesState):
    pass

# Graph
builder = StateGraph(MessagesState)

# Define nodes: these do the work
builder.add_node("assistant", assistant)
builder.add_node("tools", ToolNode(tools))
builder.add_node("human_feedback", human_feedback)

# Define edges: these determine the control flow
builder.add_edge(START, "human_feedback")
builder.add_edge("human_feedback", "assistant")
builder.add_conditional_edges(
    "assistant",
    tools_condition,
)
builder.add_edge("tools", "human_feedback")

memory = MemorySaver()
graph = builder.compile(interrupt_before=["human_feedback"], checkpointer=memory)
display(Image(graph.get_graph().draw_mermaid_png()))
```

该图的结构大致如下：

![image](D:\Majinliang\Documents\笔记\前沿技术\Agent\Inbox\image.png)

当获取到用户反馈后，使用 `.update_state` 使用获取到的人工响应以更新图的状态。在 `update_state` 方法中，使用 `as_node="human_feedback"` 参数将状态更新作用于指定节点。

`as_node` 参数的作用是，将本次的手动状态更新作为**指定节点**的输出，使 LangGraph 可以从该节点之后继续执行。

```python
# Get user input
user_input = input("Tell me how you want to update the state: ")

# We now update the state as if we are the human_feedback node
graph.update_state(thread, {"messages": user_input}, as_node="human_feedback")

# Continue the graph execution
for event in graph.stream(None, thread, stream_mode="values"):
    event["messages"][-1].pretty_print()
```


# 动态断点

在先前，断点协助我们实现了消息追加、消息修改和人工干预获取用户输入等功能。然而，先前的断点都只能在图编译期间由开发者针对某一个特定的节点设置。

## 普通动态断点

通过 NodeInterrupt 可以实现图自身的动态中断。通过动态中断，可以实现：
- 在节点内部，通过开发者定义的逻辑，有条件的触发断点；
- 通过向 NodeInterrupt 中写入内容，向用户说明断点触发原因。

下面是一个根据用户传入消息动态中断的示例：

```python
def step_2(state: State) -> State:
    # 当输入长度 > 5 时, 向用户抛出 NodeInterrupt
    if len(state['input']) > 5:
        raise NodeInterrupt(f"Received input that is longer than 5 characters: {state['input']}")
    
    return state

builder = StateGraph(State)

# Compile the graph with memory
graph = builder.compile(checkpointer=memory)
```

当用户传入的消息长度大于 5 时，向用户抛出异常并中断图流程。中断后，图的下一个待执行节点仍为当前节点，`Interrupt` 会被记录到图状态中。

当尝试传入 `None` 以从断点处恢复图执行时，除非图的状态发生变化，否则会一直卡在 `Interrupt` 节点。

对于 API，也是类似的方式。

# Time Travel


