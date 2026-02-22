# LangGraph Patterns — Compact Reference
*8 patterns for custom graph-based agents. When/where to use each one.*
*Full examples: agent-langgraph skill | Last updated: 2026-02-22*

---

## LG1 — StateGraph + compile

**What:** Define graph topology: nodes + edges → compile → runnable.
**When:** Every custom LangGraph agent. Always the outer wrapper.
**Where:** Top-level of every agent file.
**Key rule:** `builder.add_edge(START, "first_node")` is always required.
Nodes return partial state dicts — only updated keys, never the full state.

```python
from langgraph.graph import StateGraph, START, END
from typing_extensions import TypedDict

class State(TypedDict):
    topic: str
    result: str

def process(state: State):
    return {"result": f"Done: {state['topic']}"}

builder = StateGraph(State)
builder.add_node("process", process)
builder.add_edge(START, "process")
builder.add_edge("process", END)
graph = builder.compile()

result = graph.invoke({"topic": "LangGraph", "result": ""})
```

---

## LG2 — MessagesState

**What:** Prebuilt state with `messages: list[AnyMessage]` + `add_messages` reducer.
**When:** Any agent that uses an LLM — the model sends/receives messages.
**Where:** Replace custom TypedDict when you only need message history + extras.
**Key rule:** Extend it to add custom fields. Never overwrite `messages` — always append.

```python
from langgraph.graph import MessagesState

# Use as-is for pure message history
class State(MessagesState):
    user_name: str   # add extra fields here

# nodes return {"messages": [response]} — reducer appends automatically
```

---

## LG3 — ToolNode + tools_condition

**What:** Prebuilt node that executes tool calls from the last AIMessage.
**When:** Every ReAct-style agent (LLM → tools → LLM loop).
**Where:** Pair with `tools_condition` on conditional edges from the LLM node.
**Key rule:** Bind the same `tools` list to both the model and `ToolNode`.

```python
from langgraph.prebuilt import ToolNode, tools_condition
from langchain.chat_models import init_chat_model
from langchain.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

tools = [multiply]
model = init_chat_model("claude-sonnet-4-5-20250929").bind_tools(tools)

def call_llm(state: MessagesState):
    return {"messages": [model.invoke(state["messages"])]}

builder = StateGraph(MessagesState)
builder.add_node("llm", call_llm)
builder.add_node("tools", ToolNode(tools))
builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", tools_condition)  # → "tools" or END
builder.add_edge("tools", "llm")
graph = builder.compile()
```

---

## LG4 — Checkpointer (multi-turn memory)

**What:** Persists graph state between invocations. Enables multi-turn conversations.
**When:** Any agent that needs to remember past turns (assistant, booking agent).
**Where:** `builder.compile(checkpointer=...)`. Always pass `thread_id` in config.
**Key rule:** Same `thread_id` = same conversation. Different `thread_id` = new session.

```python
from langgraph.checkpoint.memory import InMemorySaver      # dev/testing
# from langgraph.checkpoint.sqlite import SqliteSaver      # local persistence
# from langgraph.checkpoint.postgres import PostgresSaver  # production

checkpointer = InMemorySaver()
graph = builder.compile(checkpointer=checkpointer)

config = {"configurable": {"thread_id": "user-123"}}
graph.invoke({"messages": [{"role": "user", "content": "I'm Alice"}]}, config)
graph.invoke({"messages": [{"role": "user", "content": "What's my name?"}]}, config)
# → "Alice" — state was remembered
```

---

## LG5 — Command (state update + routing)

**What:** Return state update AND routing decision from a single node.
**When:** Node needs to both update state AND decide the next node (no separate conditional edge needed).
**Where:** Any node with conditional routing logic. Replaces `add_conditional_edges` when routing lives in the node.
**Key rule:** Annotate return type `Command[Literal["node_a", "node_b"]]` — required for graph rendering.

```python
from langgraph.types import Command
from typing import Literal

def classify(state: State) -> Command[Literal["handle_bug", "handle_question"]]:
    if "error" in state["message"].lower():
        return Command(
            update={"category": "bug"},
            goto="handle_bug"
        )
    return Command(
        update={"category": "question"},
        goto="handle_question"
    )
# No conditional edge needed — routing is inside the node
```

---

## LG6 — Conditional edges

**What:** Route to different nodes based on state. Separate routing function from node logic.
**When:** Routing logic is simple or reusable. LLM tool-call check (use `tools_condition`).
**Where:** `builder.add_conditional_edges("node", router_fn, {"a": "node_a", "b": "node_b"})`.
**Key rule:** Router returns a string key. Map string → node name in the dict.

```python
from typing import Literal

def router(state: State) -> Literal["tools", "__end__"]:
    last = state["messages"][-1]
    if last.tool_calls:
        return "tools"
    return END

builder.add_conditional_edges("llm", router, {"tools": "tools", END: END})
# Or use prebuilt: builder.add_conditional_edges("llm", tools_condition)
```

---

## LG7 — Async nodes

**What:** Async node functions for non-blocking I/O inside the graph.
**When:** Any node that calls an async API, DB, or uses `await`.
**Where:** Node functions — change `def` → `async def`, `invoke` → `ainvoke`.
**Key rule:** All nodes must be consistently sync or async. Use `await graph.ainvoke()`.

```python
async def call_llm(state: MessagesState):
    response = await model.ainvoke(state["messages"])
    return {"messages": [response]}

# Run the graph
result = await graph.ainvoke({"messages": [...]}, config)

# Stream tokens
async for chunk, metadata in graph.astream(input, config, stream_mode="messages"):
    if chunk.content:
        print(chunk.content, end="", flush=True)
```

---

## LG8 — Subgraphs

**What:** Compile a graph as a node inside another graph. Modular multi-agent flows.
**When:** Breaking a complex agent into independent sub-agents. Reusable graph components.
**Where:** `parent_builder.add_node("sub", subgraph.compile())`.
**Key rule:** Subgraph state must share at least one key with parent state (the handoff channel).

```python
# Build subgraph
sub_builder = StateGraph(SubState)
sub_builder.add_node("step", do_step)
sub_builder.add_edge(START, "step")
sub_builder.add_edge("step", END)
subgraph = sub_builder.compile()

# Use as a node in parent graph
parent_builder = StateGraph(ParentState)
parent_builder.add_node("research", subgraph)   # subgraph is a node
parent_builder.add_node("summarize", summarize)
parent_builder.add_edge(START, "research")
parent_builder.add_edge("research", "summarize")
parent = parent_builder.compile()
```

---

## Quick Decision Table

| Situation | Use |
|-----------|-----|
| Build any custom agent | LG1 StateGraph + compile |
| Agent uses LLM messages | LG2 MessagesState (or Annotated[list, add_messages]) |
| LLM calls tools in a loop | LG3 ToolNode + tools_condition |
| Agent remembers past turns | LG4 Checkpointer + thread_id |
| Node updates state AND routes | LG5 Command(update=, goto=) |
| Simple routing after a node | LG6 add_conditional_edges |
| Nodes make async API calls | LG7 async def + ainvoke |
| Multi-agent / modular flow | LG8 Subgraphs as nodes |
