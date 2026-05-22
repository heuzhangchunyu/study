# LangGraph 企业级 AI Agent 开发学习笔记

> 适用对象：已经了解 LangChain 基础，想进一步开发可控、可恢复、可观测、可部署的企业级 Agent 工作流的 Python 工程师。  
> 版本说明：本文按 LangGraph Python v1.x 主线整理，重点覆盖 `StateGraph`、节点、边、条件路由、状态管理、checkpoint、interrupt、streaming、durable execution、subgraph、prebuilt agent 与生产化实践。  
> 学习目标：读完后能独立设计一个支持多步骤推理、工具调用、人机审批、状态持久化、失败恢复和线上观测的企业级 AI Agent。

## 目录

1. [LangGraph 是什么](#langgraph-是什么)
2. [LangGraph 和 LangChain 的关系](#langgraph-和-langchain-的关系)
3. [环境准备](#环境准备)
4. [核心心智模型](#核心心智模型)
5. [第一个 StateGraph](#第一个-stategraph)
6. [State 状态定义](#state-状态定义)
7. [Node 节点](#node-节点)
8. [Edge 边](#edge-边)
9. [Conditional Edge 条件路由](#conditional-edge-条件路由)
10. [Reducer 状态合并](#reducer-状态合并)
11. [MessagesState 对话状态](#messagesstate-对话状态)
12. [接入 Chat Model](#接入-chat-model)
13. [Tool 工具调用](#tool-工具调用)
14. [Prebuilt ReAct Agent](#prebuilt-react-agent)
15. [Memory 与 Checkpoint](#memory-与-checkpoint)
16. [Thread ID 会话隔离](#thread-id-会话隔离)
17. [Human-in-the-loop 人机协同](#human-in-the-loop-人机协同)
18. [Command 控制流](#command-控制流)
19. [Streaming 流式输出](#streaming-流式输出)
20. [Durable Execution 持久执行](#durable-execution-持久执行)
21. [Subgraph 子图](#subgraph-子图)
22. [RAG + LangGraph](#rag--langgraph)
23. [多 Agent 协作](#多-agent-协作)
24. [FastAPI 部署](#fastapi-部署)
25. [企业级工程结构](#企业级工程结构)
26. [测试与评估](#测试与评估)
27. [安全与合规](#安全与合规)
28. [常见业务场景方案](#常见业务场景方案)
29. [学习路线](#学习路线)
30. [上线检查清单](#上线检查清单)
31. [官方资料](#官方资料)

## LangGraph 是什么

LangGraph 是 LangChain 生态里的 **Agent 编排框架**。它的核心思想是：把 Agent 或 AI 工作流建模成一个图。

```text
状态 State
  -> 节点 Node
  -> 边 Edge
  -> 条件路由 Conditional Edge
  -> 持久化 Checkpoint
  -> 暂停/恢复 Interrupt
```

LangGraph 适合解决普通 Chain 或简单 Agent 难以稳定处理的问题：

| 能力 | 解决的问题 | 示例场景 |
| --- | --- | --- |
| StateGraph | 显式管理流程状态 | 多步骤客服、审批、工单处理 |
| 条件路由 | 根据状态决定下一步 | 分类后走不同业务流程 |
| Checkpoint | 保存每一步状态 | 多轮会话、失败恢复 |
| Interrupt | 暂停等待人类输入 | 退款审批、合同确认、发邮件前确认 |
| Streaming | 实时返回进度和 token | Chat UI、Agent 执行过程展示 |
| Durable execution | 长任务可恢复 | 复杂数据处理、长流程 Agent |
| Subgraph | 复用子流程 | RAG 子流程、审批子流程、工具调用子流程 |

一句话：

**LangChain 更像组件工具箱，LangGraph 更像可持久化的 Agent 工作流引擎。**

## LangGraph 和 LangChain 的关系

LangGraph 可以和 LangChain 一起使用，但不是必须依赖 LangChain。

### 使用场景

当你的应用只是：

```text
Prompt -> Model -> Parser
```

用 LangChain LCEL 就够了。

当你的应用变成：

```text
用户输入
  -> 判断意图
  -> 查知识库
  -> 调工具
  -> 必要时人工审批
  -> 根据审批结果继续
  -> 保存状态
  -> 支持失败恢复
```

这时应该使用 LangGraph。

### 对比

| 对比项 | LangChain | LangGraph |
| --- | --- | --- |
| 核心用途 | LLM 组件编排 | 状态化 Agent 编排 |
| 典型结构 | Chain | Graph |
| 状态管理 | 较弱，需要自己组织 | 原生 State |
| 流程控制 | 线性或简单分支 | 显式节点、边、条件路由 |
| 持久化 | 通常外部实现 | checkpoint 原生支持 |
| 人机协同 | 需要自己做 | interrupt 原生支持 |
| 适合场景 | 摘要、分类、RAG 固定链 | 长流程、多工具、多 Agent、审批流 |

## 环境准备

### 使用场景

本节用于初始化 LangGraph 项目。企业项目通常会同时使用 LangChain 模型封装、LangGraph 编排和 LangSmith 追踪。

### 安装示例

```bash
python -m venv .venv
source .venv/bin/activate

pip install -U \
  langgraph \
  langchain \
  langchain-openai \
  langchain-community \
  langchain-chroma \
  langchain-text-splitters \
  langsmith \
  fastapi \
  uvicorn \
  pydantic \
  pydantic-settings \
  python-dotenv
```

### `.env` 示例

```bash
OPENAI_API_KEY=sk-xxx
LANGSMITH_TRACING=true
LANGSMITH_API_KEY=lsv2_xxx
LANGSMITH_PROJECT=langgraph-enterprise-dev
```

### Python 读取环境变量

```python
from dotenv import load_dotenv

load_dotenv()
```

## 核心心智模型

LangGraph 的最小心智模型：

```text
State：整个流程共享的数据
Node：处理 State 的函数
Edge：决定节点执行顺序
Graph：把 State、Node、Edge 编译成可运行应用
Checkpoint：保存每一步 State
Interrupt：暂停流程，等待外部输入
```

LangGraph 节点不是直接“改全局变量”，而是返回一个字典，LangGraph 再把返回值合并进 State。

```text
当前 State
  -> Node(state)
  -> 返回 {"字段": 新值}
  -> LangGraph 合并到 State
  -> 进入下一个节点
```

### 企业开发优先级

1. 先定义清楚 State。
2. 再拆分节点职责。
3. 再设计边和条件路由。
4. 再接入模型、工具、RAG。
5. 再加 checkpoint、interrupt、streaming。
6. 最后做观测、测试、安全和部署。

## 第一个 StateGraph

### 使用场景

适合学习 LangGraph 最基本流程：输入一个状态，经过两个节点处理，输出最终状态。

### 示例：最小图

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END


class State(TypedDict):
    topic: str
    outline: str
    article: str


def make_outline(state: State):
    topic = state["topic"]
    return {"outline": f"1. 介绍{topic}\n2. 核心价值\n3. 落地建议"}


def write_article(state: State):
    return {
        "article": f"主题：{state['topic']}\n\n大纲：\n{state['outline']}\n\n正文：这里是文章正文。"
    }


builder = StateGraph(State)

builder.add_node("make_outline", make_outline)
builder.add_node("write_article", write_article)

builder.add_edge(START, "make_outline")
builder.add_edge("make_outline", "write_article")
builder.add_edge("write_article", END)

graph = builder.compile()

result = graph.invoke({"topic": "企业 AI 应用"})

print(result["article"])
```

### 解释

```python
class State(TypedDict):
```

定义图里的共享状态。

```python
builder.add_node("make_outline", make_outline)
```

注册一个节点。

```python
builder.add_edge("make_outline", "write_article")
```

定义节点之间的执行顺序。

```python
graph.invoke(...)
```

执行图。

## State 状态定义

State 是 LangGraph 应用的核心。它定义整个流程中有哪些字段。

### 使用场景

适合需要在多个节点之间传递数据的流程，例如问题、分类结果、检索文档、工具结果、审批结果、最终答案。

### 示例：客服工单 State

```python
from typing import TypedDict, Literal


class TicketState(TypedDict):
    user_input: str
    category: Literal["bug", "question", "refund", "other"]
    priority: Literal["low", "medium", "high", "urgent"]
    order_id: str | None
    retrieved_context: str
    draft_answer: str
    approved: bool
    final_answer: str
```

### 示例：RAG State

```python
from typing import TypedDict
from langchain_core.documents import Document


class RAGState(TypedDict):
    question: str
    documents: list[Document]
    context: str
    answer: str
```

### 企业建议

- State 字段命名要业务化，不要全是 `data`、`result`。
- 节点只返回自己负责更新的字段。
- 高风险字段要显式建模，例如 `approved`、`risk_level`、`human_comment`。
- 不要把大对象无限塞入 State，长文本和文件应存外部存储，只在 State 放引用。

## Node 节点

Node 是普通 Python 函数：接收 State，返回要更新的字段。

### 使用场景

适合把业务流程拆成独立步骤，例如分类、检索、生成、审核、调用工具、保存结果。

### 示例：分类节点

```python
def classify_ticket(state: TicketState):
    text = state["user_input"]

    if "退款" in text:
        category = "refund"
    elif "无法登录" in text or "报错" in text:
        category = "bug"
    elif "怎么" in text or "如何" in text:
        category = "question"
    else:
        category = "other"

    return {"category": category}
```

### 示例：生成回复节点

```python
def generate_answer(state: TicketState):
    if state["category"] == "refund":
        answer = "请提供订单号，我会帮您查询退款条件。"
    elif state["category"] == "bug":
        answer = "已记录系统问题，请补充报错截图和发生时间。"
    else:
        answer = "我会根据您的问题继续处理。"

    return {"draft_answer": answer}
```

### 企业建议

- 一个节点只做一件事。
- 节点要可测试，尽量不要依赖全局状态。
- 外部 API 调用要加超时、重试、异常兜底。
- 有副作用的节点要设计幂等性，例如创建工单不要重复创建。

## Edge 边

Edge 决定节点执行顺序。

### 使用场景

适合固定流程，例如“分类 -> 检索 -> 生成 -> 审核 -> 返回”。

### 示例：固定边

```python
from langgraph.graph import StateGraph, START, END

builder = StateGraph(TicketState)

builder.add_node("classify", classify_ticket)
builder.add_node("generate", generate_answer)

builder.add_edge(START, "classify")
builder.add_edge("classify", "generate")
builder.add_edge("generate", END)

graph = builder.compile()
```

### 企业建议

- 固定流程用普通 edge。
- 有分支判断时用 conditional edge。
- 节点名建议使用动词短语，例如 `classify_ticket`、`retrieve_docs`、`request_approval`。

## Conditional Edge 条件路由

条件路由根据 State 决定下一个节点。

### 使用场景

适合意图分流、风险分级、是否调用工具、是否人工审批。

### 示例：根据工单类型路由

```python
from langgraph.graph import StateGraph, START, END


def route_by_category(state: TicketState) -> str:
    if state["category"] == "refund":
        return "handle_refund"
    if state["category"] == "bug":
        return "handle_bug"
    return "handle_general"


def handle_refund(state: TicketState):
    return {"draft_answer": "退款问题需要先查询订单状态。"}


def handle_bug(state: TicketState):
    return {"draft_answer": "技术问题已进入故障处理流程。"}


def handle_general(state: TicketState):
    return {"draft_answer": "这是通用问题回复。"}


builder = StateGraph(TicketState)

builder.add_node("classify", classify_ticket)
builder.add_node("handle_refund", handle_refund)
builder.add_node("handle_bug", handle_bug)
builder.add_node("handle_general", handle_general)

builder.add_edge(START, "classify")
builder.add_conditional_edges(
    "classify",
    route_by_category,
    {
        "handle_refund": "handle_refund",
        "handle_bug": "handle_bug",
        "handle_general": "handle_general",
    },
)
builder.add_edge("handle_refund", END)
builder.add_edge("handle_bug", END)
builder.add_edge("handle_general", END)

graph = builder.compile()
```

### 企业建议

- 路由函数要简单、确定、容易测试。
- 路由返回值建议用枚举或固定字符串。
- 路由逻辑可以由模型判断，但高风险流程建议用程序规则二次校验。

## Reducer 状态合并

默认情况下，节点返回的新值会覆盖 State 里的旧值。有些字段需要“追加”，例如消息列表、日志列表、步骤记录，这时要用 reducer。

### 使用场景

适合聊天消息、执行轨迹、检索文档列表、工具调用记录。

### 示例：列表追加

```python
from typing import Annotated, TypedDict
import operator


class LogState(TypedDict):
    input: str
    steps: Annotated[list[str], operator.add]


def step_one(state: LogState):
    return {"steps": ["完成第一步"]}


def step_two(state: LogState):
    return {"steps": ["完成第二步"]}
```

如果两个节点分别返回：

```python
{"steps": ["完成第一步"]}
{"steps": ["完成第二步"]}
```

最终 State 里的 `steps` 会合并成：

```python
["完成第一步", "完成第二步"]
```

### 企业建议

- 日志、消息、工具轨迹用 reducer。
- 业务最终结果通常用覆盖。
- reducer 字段要控制长度，避免状态越来越大。

## MessagesState 对话状态

LangGraph 提供了适合聊天场景的 `MessagesState`，它内置消息列表合并逻辑。

### 使用场景

适合多轮聊天、客服助手、Agent 对话。

### 示例：聊天节点

```python
from langchain.chat_models import init_chat_model
from langgraph.graph import StateGraph, START, END, MessagesState

model = init_chat_model("gpt-4.1-mini", model_provider="openai")


def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}


builder = StateGraph(MessagesState)
builder.add_node("call_model", call_model)
builder.add_edge(START, "call_model")
builder.add_edge("call_model", END)

graph = builder.compile()

result = graph.invoke({
    "messages": [{"role": "user", "content": "什么是 LangGraph？"}]
})

print(result["messages"][-1].content)
```

### 企业建议

- 聊天类应用优先使用 `MessagesState`。
- 消息历史不能无限增长，需要摘要或裁剪。
- 会话隔离要配合 `thread_id` 和 checkpoint。

## 接入 Chat Model

LangGraph 节点里可以直接调用 LangChain 的模型。

### 使用场景

适合模型分类、摘要、生成、判断、路由。

### 示例：模型分类节点

```python
from typing import Literal
from pydantic import BaseModel, Field
from langchain.chat_models import init_chat_model


class Classification(BaseModel):
    category: Literal["refund", "bug", "question", "other"] = Field(
        description="用户问题分类"
    )
    priority: Literal["low", "medium", "high", "urgent"] = Field(
        description="问题优先级"
    )


model = init_chat_model("gpt-4.1-mini", model_provider="openai", temperature=0)
structured_model = model.with_structured_output(Classification)


def classify_with_model(state: TicketState):
    result = structured_model.invoke(
        f"请分类这个客服问题：{state['user_input']}"
    )
    return {
        "category": result.category,
        "priority": result.priority,
    }
```

### 企业建议

- 分类、路由类模型调用建议 temperature=0。
- 结构化输出比自然语言更适合驱动流程。
- 模型判断后可加规则兜底，避免高风险误路由。

## Tool 工具调用

LangGraph 可以手写工具节点，也可以使用 prebuilt tool node 或 prebuilt agent。

### 使用场景

适合调用订单系统、库存系统、CRM、数据库、搜索服务、工单系统。

### 示例：手写工具节点

```python
def query_order_api(order_id: str) -> str:
    fake_orders = {
        "A1001": "已付款，仓库拣货中",
        "A1002": "已发货，预计明天送达",
    }
    return fake_orders.get(order_id, "未找到订单")


def query_order_node(state: TicketState):
    order_id = state.get("order_id")
    if not order_id:
        return {"draft_answer": "请提供订单号。"}

    status = query_order_api(order_id)
    return {"retrieved_context": f"订单 {order_id} 状态：{status}"}
```

### 示例：工具函数

```python
from langchain.tools import tool


@tool
def get_order_status(order_id: str) -> str:
    """根据订单 ID 查询订单状态。"""
    return query_order_api(order_id)
```

### 企业建议

- 简单可控流程可以手写工具节点。
- 开放式工具选择可以用 prebuilt agent。
- 工具内部必须鉴权，不能只依赖模型判断。
- 破坏性工具必须加人工审批。

## Prebuilt ReAct Agent

LangGraph 提供了预构建 ReAct Agent，适合快速构建“模型 + 工具”的 Agent。

### 使用场景

适合智能客服、查询助手、运维助手、数据分析助手。

### 示例：订单查询 Agent

```python
from langgraph.prebuilt import create_react_agent
from langchain.tools import tool


@tool
def get_order_status(order_id: str) -> str:
    """根据订单 ID 查询订单状态。"""
    fake_orders = {
        "A1001": "已付款，仓库拣货中",
        "A1002": "已发货，预计明天送达",
    }
    return fake_orders.get(order_id, "未找到订单")


agent = create_react_agent(
    model="gpt-4.1-mini",
    tools=[get_order_status],
    prompt="你是电商客服助手，回答要简洁。"
)

result = agent.invoke({
    "messages": [
        {"role": "user", "content": "帮我查一下订单 A1002"}
    ]
})

print(result["messages"][-1].content)
```

### 企业建议

- 快速验证用 `create_react_agent`。
- 复杂生产流程用自定义 `StateGraph`。
- 工具数量不要太多，先按业务域拆分 Agent。

## Memory 与 Checkpoint

Checkpoint 是 LangGraph 的持久化机制。它会保存图执行过程中的状态，让同一个 thread 可以继续对话或恢复执行。

### 使用场景

适合多轮对话、长任务恢复、人机审批、失败重试、状态审计。

### 示例：内存 checkpoint

```python
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.graph import StateGraph, START, END, MessagesState
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-4.1-mini", model_provider="openai")
checkpointer = InMemorySaver()


def call_model(state: MessagesState):
    response = model.invoke(state["messages"])
    return {"messages": [response]}


builder = StateGraph(MessagesState)
builder.add_node("call_model", call_model)
builder.add_edge(START, "call_model")
builder.add_edge("call_model", END)

graph = builder.compile(checkpointer=checkpointer)
```

### 企业建议

- `InMemorySaver` 只适合开发和测试。
- 生产应使用持久化 checkpointer，例如数据库方案。
- 每个用户会话必须带稳定 `thread_id`。
- checkpoint 中可能包含敏感信息，要注意存储安全和脱敏。

## Thread ID 会话隔离

`thread_id` 是 LangGraph 识别一次会话或一次流程实例的关键。

### 使用场景

适合同一个用户多轮对话、同一个审批流程多次恢复、同一个任务失败后重试。

### 示例：同一 thread 保留上下文

```python
config = {
    "configurable": {
        "thread_id": "user-001-session-001"
    }
}

graph.invoke(
    {"messages": [{"role": "user", "content": "我喜欢简短回答。"}]},
    config=config,
)

result = graph.invoke(
    {"messages": [{"role": "user", "content": "解释一下 RAG。"}]},
    config=config,
)

print(result["messages"][-1].content)
```

### 企业建议

- thread_id 不要直接使用可猜测的用户 ID。
- 建议格式：`tenant_id:user_id:conversation_id` 的安全哈希或 UUID。
- 不同租户必须隔离。
- thread 生命周期要可控，过期会话应清理。

## Human-in-the-loop 人机协同

`interrupt()` 可以让图执行到某个节点时暂停，等待人类输入，再继续执行。

### 使用场景

适合退款审批、合同发送前确认、自动发邮件前确认、数据库写入前确认、高风险操作复核。

### 示例：审批节点

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.types import interrupt, Command


class ApprovalState(TypedDict):
    request: str
    approved: bool
    result: str


def prepare_action(state: ApprovalState):
    return {"result": f"准备执行操作：{state['request']}"}


def approval_node(state: ApprovalState):
    approved = interrupt({
        "question": "是否批准执行该操作？",
        "request": state["request"],
    })
    return {"approved": approved}


def execute_action(state: ApprovalState):
    if state["approved"]:
        return {"result": "操作已执行"}
    return {"result": "操作已取消"}


builder = StateGraph(ApprovalState)
builder.add_node("prepare_action", prepare_action)
builder.add_node("approval", approval_node)
builder.add_node("execute_action", execute_action)

builder.add_edge(START, "prepare_action")
builder.add_edge("prepare_action", "approval")
builder.add_edge("approval", "execute_action")
builder.add_edge("execute_action", END)

graph = builder.compile(checkpointer=InMemorySaver())

config = {"configurable": {"thread_id": "approval-001"}}

first_result = graph.invoke(
    {"request": "为订单 A1002 发起退款", "approved": False, "result": ""},
    config=config,
)

print(first_result["__interrupt__"])

second_result = graph.invoke(
    Command(resume=True),
    config=config,
)

print(second_result["result"])
```

### 企业建议

- interrupt payload 必须是 JSON 可序列化数据。
- interrupt 前的副作用必须幂等。
- 不要把 `interrupt()` 包在 `try/except` 里吞掉。
- 审批结果要写审计日志。

## Command 控制流

`Command` 可以用于恢复 interrupt，也可以表达图中的控制动作。

### 使用场景

适合恢复人机审批、动态跳转、更新状态并指定下一步。

### 示例：恢复 interrupt

```python
from langgraph.types import Command

result = graph.invoke(
    Command(resume=False),
    config={"configurable": {"thread_id": "approval-001"}},
)
```

这里 `resume=False` 会作为 `interrupt()` 的返回值回到被暂停的节点。

### 企业建议

- 人机审批恢复必须使用同一个 `thread_id`。
- 前端保存 interrupt 信息和 thread_id，审批完成后再 resume。
- 恢复时要校验当前用户是否有审批权限。

## Streaming 流式输出

LangGraph 支持流式返回节点更新、消息 token、调试信息等。

### 使用场景

适合 Chat UI、Agent 执行进度、长任务状态展示、工具调用过程展示。

### 示例：流式节点更新

```python
for chunk in graph.stream(
    {"messages": [{"role": "user", "content": "什么是 LangGraph？"}]},
    stream_mode="updates",
):
    print(chunk)
```

### 示例：流式消息

```python
for chunk in graph.stream(
    {"messages": [{"role": "user", "content": "写一段项目总结"}]},
    stream_mode="messages",
):
    print(chunk)
```

### 企业建议

- UI 里区分节点进度和模型 token。
- SSE 或 WebSocket 都可以承载流式输出。
- 流式返回不代表不用保存最终结果，生产仍要落库。

## Durable Execution 持久执行

Durable execution 指工作流每一步能保存状态，进程中断后可以从 checkpoint 恢复。

### 使用场景

适合长流程 Agent、人工审批、多步骤数据处理、外部 API 不稳定的任务。

### 示例：指定 durability

```python
for update in graph.stream(
    {"input": "开始处理任务"},
    config={"configurable": {"thread_id": "job-001"}},
    durability="sync",
):
    print(update)
```

### durability 模式

| 模式 | 含义 | 适合场景 |
| --- | --- | --- |
| `exit` | 结束或中断时保存 | 性能优先，能接受中途崩溃丢进度 |
| `async` | 异步保存 checkpoint | 通用生产场景 |
| `sync` | 每步前同步保存 | 高可靠流程，例如审批、资金、合同 |

### 企业建议

- 高风险业务用 `sync`。
- 大吞吐低风险任务可用 `async`。
- 节点内调用外部系统要配合幂等键。
- 恢复执行时使用相同 thread_id。

## Subgraph 子图

Subgraph 是把一个图作为另一个图里的节点或流程复用。

### 使用场景

适合复用复杂子流程，例如 RAG 子图、审批子图、合同审查子图、订单处理子图。

### 示例：把 RAG 做成子图

```python
from typing import TypedDict
from langgraph.graph import StateGraph, START, END


class RagSubState(TypedDict):
    question: str
    context: str
    answer: str


def retrieve(state: RagSubState):
    return {"context": "这里是检索到的知识库内容"}


def generate(state: RagSubState):
    return {"answer": f"基于上下文回答：{state['question']}"}


rag_builder = StateGraph(RagSubState)
rag_builder.add_node("retrieve", retrieve)
rag_builder.add_node("generate", generate)
rag_builder.add_edge(START, "retrieve")
rag_builder.add_edge("retrieve", "generate")
rag_builder.add_edge("generate", END)

rag_graph = rag_builder.compile()
```

### 在主图中调用

```python
def call_rag_subgraph(state):
    result = rag_graph.invoke({"question": state["user_input"], "context": "", "answer": ""})
    return {"draft_answer": result["answer"]}
```

### 企业建议

- 可复用流程优先封装成 subgraph。
- 父图和子图的 State 字段要明确映射。
- 需要持久化 interrupt 时，父图也要配置 checkpointer。

## RAG + LangGraph

LangGraph 非常适合把 RAG 拆成显式流程。

### 使用场景

适合企业知识库问答、文档审查、客服知识助手。

### 流程设计

```text
用户问题
  -> rewrite_question
  -> retrieve_docs
  -> grade_docs
  -> generate_answer
  -> verify_answer
  -> final
```

### 示例：RAG 图

```python
from typing import TypedDict
from langchain_core.documents import Document
from langchain.chat_models import init_chat_model
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langgraph.graph import StateGraph, START, END


class RAGState(TypedDict):
    question: str
    documents: list[Document]
    context: str
    answer: str


model = init_chat_model("gpt-4.1-mini", model_provider="openai", temperature=0)

prompt = ChatPromptTemplate.from_template("""
你是企业知识库助手。
只能根据上下文回答。

上下文：
{context}

问题：
{question}
""")

chain = prompt | model | StrOutputParser()


def retrieve_docs(state: RAGState):
    docs = retriever.invoke(state["question"])
    return {"documents": docs}


def format_context(state: RAGState):
    context = "\n\n".join(
        f"[来源: {doc.metadata.get('source', 'unknown')}]\n{doc.page_content}"
        for doc in state["documents"]
    )
    return {"context": context}


def generate_answer(state: RAGState):
    answer = chain.invoke({
        "context": state["context"],
        "question": state["question"],
    })
    return {"answer": answer}


builder = StateGraph(RAGState)
builder.add_node("retrieve_docs", retrieve_docs)
builder.add_node("format_context", format_context)
builder.add_node("generate_answer", generate_answer)

builder.add_edge(START, "retrieve_docs")
builder.add_edge("retrieve_docs", "format_context")
builder.add_edge("format_context", "generate_answer")
builder.add_edge("generate_answer", END)

rag_graph = builder.compile()
```

### 企业建议

- RAG 固定链路用 LangChain 即可；需要多步骤验证、重试、人工复核时用 LangGraph。
- 可以增加 `grade_docs` 节点判断检索结果是否足够。
- 可以增加 `fallback` 节点处理无答案场景。
- 可以增加 `human_review` 节点处理高风险问题。

## 多 Agent 协作

LangGraph 可以把多个 Agent 或多个专业节点编排在一个图里。

### 使用场景

适合复杂任务分工，例如“研究员 Agent -> 分析师 Agent -> 审核员 Agent -> 写作 Agent”。

### 示例：简单多角色流程

```python
class ReportState(TypedDict):
    topic: str
    research_notes: str
    analysis: str
    final_report: str


def researcher(state: ReportState):
    return {"research_notes": f"关于 {state['topic']} 的资料摘要"}


def analyst(state: ReportState):
    return {"analysis": f"基于资料的分析：{state['research_notes']}"}


def writer(state: ReportState):
    return {"final_report": f"最终报告：{state['analysis']}"}


builder = StateGraph(ReportState)
builder.add_node("researcher", researcher)
builder.add_node("analyst", analyst)
builder.add_node("writer", writer)

builder.add_edge(START, "researcher")
builder.add_edge("researcher", "analyst")
builder.add_edge("analyst", "writer")
builder.add_edge("writer", END)

graph = builder.compile()
```

### 企业建议

- 多 Agent 不等于越多越好。
- 先把任务拆成稳定节点，再决定哪些节点需要模型。
- 每个 Agent 的输入输出 schema 要明确。
- 增加审核节点，避免错误在多个 Agent 间放大。

## FastAPI 部署

LangGraph 图可以作为服务对象挂到 FastAPI。

### 使用场景

适合把 Agent 工作流暴露给前端、企业微信、飞书、CRM、工单系统。

### 示例：普通接口

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI(title="LangGraph Agent API")


class ChatRequest(BaseModel):
    thread_id: str
    message: str


@app.post("/chat")
def chat(req: ChatRequest):
    result = graph.invoke(
        {"messages": [{"role": "user", "content": req.message}]},
        config={"configurable": {"thread_id": req.thread_id}},
    )
    return {
        "answer": result["messages"][-1].content
    }
```

### 示例：流式接口 SSE

```python
from fastapi.responses import StreamingResponse


@app.post("/chat/stream")
def chat_stream(req: ChatRequest):
    def event_generator():
        for chunk in graph.stream(
            {"messages": [{"role": "user", "content": req.message}]},
            config={"configurable": {"thread_id": req.thread_id}},
            stream_mode="updates",
        ):
            yield f"data: {chunk}\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

### 企业建议

- graph 应在应用启动时初始化。
- HTTP 请求里传 `thread_id`，后端校验归属。
- SSE 断开时要处理任务状态。
- 对 interrupt 的恢复设计单独接口。

## 企业级工程结构

推荐结构：

```text
langgraph_app/
  app/
    api/
      routes_chat.py
      routes_approval.py
    core/
      config.py
      logging.py
      security.py
    graphs/
      customer_service_graph.py
      rag_graph.py
      approval_graph.py
    nodes/
      classify.py
      retrieve.py
      generate.py
      approve.py
      tools.py
    schemas/
      state.py
      requests.py
      responses.py
    services/
      order_service.py
      vector_service.py
      audit_service.py
    tests/
      test_routes.py
      test_graphs.py
      test_nodes.py
  docs/
  .env
  pyproject.toml
```

### State 集中定义示例

```python
from typing import TypedDict, Literal
from langchain_core.documents import Document


class CustomerServiceState(TypedDict):
    user_id: str
    tenant_id: str
    user_input: str
    category: Literal["refund", "bug", "question", "other"]
    documents: list[Document]
    tool_result: str
    approved: bool
    final_answer: str
```

### 图工厂示例

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import InMemorySaver


def build_customer_service_graph(checkpointer=None):
    builder = StateGraph(CustomerServiceState)

    builder.add_node("classify", classify_node)
    builder.add_node("retrieve", retrieve_node)
    builder.add_node("generate", generate_node)

    builder.add_edge(START, "classify")
    builder.add_edge("classify", "retrieve")
    builder.add_edge("retrieve", "generate")
    builder.add_edge("generate", END)

    return builder.compile(checkpointer=checkpointer or InMemorySaver())
```

### 企业建议

- State、Node、Graph、API 分层。
- 节点函数独立测试。
- 外部系统调用放 service 层。
- 图构建用工厂函数，方便测试注入 mock。

## 测试与评估

LangGraph 应用要同时测试节点、路由和完整图。

### 使用场景

适合上线前验收、流程变更回归、模型替换评估。

### 示例：测试节点

```python
def test_classify_ticket_refund():
    state = {
        "user_input": "我要申请退款",
        "category": "other",
        "priority": "low",
        "order_id": None,
        "retrieved_context": "",
        "draft_answer": "",
        "approved": False,
        "final_answer": "",
    }

    result = classify_ticket(state)

    assert result["category"] == "refund"
```

### 示例：测试完整图

```python
def test_graph_basic_flow():
    result = graph.invoke({
        "user_input": "系统无法登录",
        "category": "other",
        "priority": "low",
        "order_id": None,
        "retrieved_context": "",
        "draft_answer": "",
        "approved": False,
        "final_answer": "",
    })

    assert result["category"] == "bug"
    assert result["draft_answer"]
```

### 企业建议

- 纯规则节点用单元测试。
- 模型节点用固定测试集评估。
- 条件路由必须覆盖所有分支。
- interrupt 流程要测试暂停和恢复。
- checkpoint 恢复要做集成测试。

## 安全与合规

LangGraph 常用于更复杂、更有行动能力的 Agent，因此安全更重要。

### 使用场景

适合所有会调用工具、访问企业数据、执行外部动作的 Agent。

### 工具权限校验示例

```python
def refund_node(state: CustomerServiceState):
    if not user_can_refund(state["user_id"], state["tenant_id"]):
        return {"final_answer": "您没有权限执行退款操作。"}

    if not state["approved"]:
        return {"final_answer": "退款操作需要审批。"}

    result = refund_service.refund_order(state["order_id"])
    return {"tool_result": result}
```

### 高风险操作审批示例

```python
def require_approval_for_refund(state: CustomerServiceState):
    approved = interrupt({
        "action": "refund",
        "order_id": state["order_id"],
        "amount": state.get("amount", 0),
    })
    return {"approved": approved}
```

### 企业安全清单

- 每个工具节点都要鉴权。
- 每个外部副作用都要幂等。
- 高风险动作必须 interrupt 人工确认。
- checkpoint 中的敏感字段要加密或脱敏。
- thread_id 要防止越权访问。
- Prompt 注入不能绕过工具权限。
- 审批、执行、失败恢复都要写审计日志。

## 常见业务场景方案

### 场景一：智能客服工单流转

推荐图：

```text
classify
  -> route_by_category
  -> retrieve_policy / query_order / create_ticket
  -> generate_answer
  -> human_review 可选
  -> final
```

适合 LangGraph，因为客服问题路径不固定，还需要工具和审批。

### 场景二：退款审批 Agent

推荐图：

```text
extract_order_id
  -> query_order
  -> check_refund_policy
  -> interrupt 等待人工审批
  -> execute_refund
  -> notify_user
```

### 场景三：企业知识库问答增强版

推荐图：

```text
rewrite_question
  -> retrieve
  -> grade_relevance
  -> generate
  -> verify_citations
  -> fallback_or_final
```

### 场景四：合同审查

推荐图：

```text
split_contract
  -> extract_clauses
  -> identify_risks
  -> human_legal_review
  -> generate_report
```

### 场景五：数据分析助手

推荐图：

```text
understand_question
  -> choose_metric_tool
  -> fetch_data
  -> analyze
  -> ask_followup_or_report
```

注意：不要让模型直接执行任意 SQL，应提供受控指标工具。

## 学习路线

### 第 1 阶段：基础图

目标：会定义 State、Node、Edge。

练习：

- 写一个两节点文章生成图。
- 写一个客服分类图。
- 写一个固定 RAG 图。

### 第 2 阶段：条件路由

目标：会根据状态选择不同路径。

练习：

- 工单按类型分流。
- 高优先级问题进入人工审核。
- 无检索结果进入 fallback。

### 第 3 阶段：模型与工具

目标：把 LLM、Tool、RAG 接入节点。

练习：

- 模型结构化分类。
- 订单查询工具节点。
- RAG 检索生成节点。

### 第 4 阶段：记忆与恢复

目标：掌握 checkpoint 和 thread_id。

练习：

- 多轮聊天保持上下文。
- 同一个任务中断后恢复。
- 查看 checkpoint 中状态变化。

### 第 5 阶段：人机协同与生产化

目标：会做审批、流式输出、部署和测试。

练习：

- 退款前 interrupt 审批。
- SSE 展示执行进度。
- FastAPI 暴露图服务。
- LangSmith 追踪执行链路。

## 上线检查清单

### 图设计

- [ ] State 字段清晰。
- [ ] 节点职责单一。
- [ ] 条件路由覆盖所有分支。
- [ ] 有 END 路径，避免无限循环。
- [ ] 高风险路径有审批节点。

### 稳定性

- [ ] 配置 checkpointer。
- [ ] 使用稳定 thread_id。
- [ ] 外部 API 有超时和重试。
- [ ] 副作用节点具备幂等性。
- [ ] durable execution 模式符合业务风险。

### 安全

- [ ] 工具节点有鉴权。
- [ ] checkpoint 敏感信息可控。
- [ ] thread_id 不可越权访问。
- [ ] Prompt 注入无法绕过权限。
- [ ] 审批与执行有审计日志。

### 质量

- [ ] 节点单元测试。
- [ ] 条件路由测试。
- [ ] interrupt 恢复测试。
- [ ] RAG 命中率评估。
- [ ] Agent 工具调用评估。

### 可观测性

- [ ] 开启 LangSmith tracing。
- [ ] 记录 request_id、tenant_id、thread_id。
- [ ] 记录节点耗时。
- [ ] 记录工具调用结果。
- [ ] 监控错误率、延迟、成本。

## 官方资料

- LangGraph Python 概览：https://docs.langchain.com/oss/python/langgraph
- Memory：https://docs.langchain.com/oss/python/langgraph/add-memory
- Persistence：https://docs.langchain.com/oss/python/langgraph/persistence
- Interrupts：https://docs.langchain.com/oss/python/langgraph/interrupts
- Streaming：https://docs.langchain.com/oss/python/langgraph/streaming
- Durable execution：https://docs.langchain.com/oss/python/langgraph/durable-execution
- Subgraphs：https://docs.langchain.com/oss/python/langgraph/use-subgraphs
- `create_react_agent` API：https://reference.langchain.com/python/langgraph.prebuilt/chat_agent_executor/create_react_agent

## 最后总结

LangGraph 的价值在于把 Agent 从“模型自由发挥”变成“显式、可控、可恢复的状态机”。

企业级 LangGraph 应用的核心是：

- 用 State 定义业务状态。
- 用 Node 拆分业务步骤。
- 用 Edge 和 Conditional Edge 控制流程。
- 用 Checkpoint 保存状态。
- 用 Thread ID 隔离会话。
- 用 Interrupt 做人机审批。
- 用 Streaming 改善体验。
- 用 Durable execution 支持长任务恢复。
- 用测试、审计、权限和观测保证上线质量。

真正落地时，建议先从一个明确流程开始，例如“客服工单分流”“退款审批”“知识库问答增强版”，把状态、路由、审批、持久化、追踪跑通后，再逐步扩展到多 Agent 协作和复杂自动化流程。
