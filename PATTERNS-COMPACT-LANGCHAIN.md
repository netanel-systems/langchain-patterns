# Python Patterns — Compact Reference
*12 patterns for LangChain/LangGraph projects. When/where to use each one.*
*Full examples: PATTERNS.md | Last updated: 2026-02-22*

---

## P1 — Classes + OOP

**What:** Blueprint → object. One class, many instances. Each holds its own data.
**When:** Encapsulate state + behaviour together. Agent, Tool, Config, Session.
**Where:** Every service class. `class Aria:`, `class MemoryStore:`.
**Key rule:** `__init__` stores data. Methods act on it. Inherit only when IS-A is true.

```python
class Aria:
    def __init__(self, name: str): self.name = name
    def greet(self) -> str: return f"I am {self.name}"
```

---

## P2 — Type Hints

**What:** Labels on function params/returns. Python ignores them; LangChain reads them.
**When:** Always. Every function, every class field.
**Where:** Tool definitions (schema), model validators, LangGraph state fields.

| Type | Use |
|------|-----|
| `str`, `int`, `bool`, `float` | primitives |
| `list[str]`, `dict[str, int]` | collections |
| `Optional[str]` | str or None |
| `Union[int, str]` | int or str |
| `Literal["a", "b"]` | exact values only |

```python
def book(pickup: str, seats: int = 1) -> str: ...
```

---

## P3 — TypedDict

**What:** Dict with declared shape. Labels only — no enforcement.
**When:** LangGraph agent state. You write it, you control it, no LLM fills it.
**Where:** `class AgentState(TypedDict)` — one per graph.
**Key rule:** Access with `state["key"]`. Combine with Annotated for reducers (→ P11).

```python
class AgentState(TypedDict):
    input: str
    output: str
    done: bool
```

---

## P4 — Pydantic BaseModel

**What:** Dict with enforcement. Crashes immediately on wrong type/value.
**When:** Tool inputs — LLM fills these, must be validated.
**Where:** `args_schema=MyModel` on every `@tool`.
**Key rule:** Access with `.field`. Use `Field(ge=1)`, `Literal`, `Optional`, `@model_validator`.

```python
class BookInput(BaseModel):
    pickup: str
    seats: int = Field(ge=1, le=6)
    language: Literal["english", "telugu"] = "english"
```

---

## P5 — Decorators + `@tool`

**What:** `@tool` wraps a function → LangChain tool. Reads name, type hints, docstring.
**When:** Every function the agent can call.
**Where:** Tool files. Register all tools in a list → pass to agent.
**Key rule:** Docstring = what agent reads to decide when to call. Make it clear.
Use `args_schema=PydanticModel` for production. Call with `.invoke({})` or `.ainvoke({})`.

```python
@tool("book_cab", args_schema=BookInput)
def book_cab(pickup: str, seats: int, language: str) -> str:
    """Book a cab from pickup. Use for any cab/ride request."""
    ...
tools = [book_cab, get_weather, send_reminder]
```

---

## P6 — async/await

**What:** Pause + wait without blocking. Run multiple tasks at the same time.
**When:** API calls, DB queries, any I/O that waits.
**Where:** Tool functions (use `.ainvoke()`), agent nodes, morning briefing.
**Key rule:** `async def` to define. `await` to call. `asyncio.gather()` to run in parallel.
`.ainvoke()` on sync `@tool` runs in thread pool (not true coroutine — still parallel).

```python
weather, news = await asyncio.gather(
    get_weather.ainvoke({"city": "Hyderabad"}),
    get_news.ainvoke({})
)
```

Timeouts: `await asyncio.wait_for(task, timeout=3)`
Errors: `asyncio.gather(..., return_exceptions=True)`

---

## P7 — try/except

**What:** Catch crashes. Return error string instead of crashing agent.
**When:** Every tool. Every API call. Every file read.
**Where:** Inside every `@tool` function body.
**Key rule:** Always two blocks: one for expected errors, one for unexpected.
Never crash the agent — always return a string.

```python
try:
    return call_api(city)
except ValueError as e:    # expected — you raised it
    return f"Error: {e}"
except Exception as e:     # unexpected — library/network
    return f"Unexpected: {e}"
```

`else:` runs only on success. `finally:` always runs (cleanup).
Custom exceptions: `class MyError(Exception): pass`

---

## P8 — Dataclasses

**What:** Auto `__init__` + `__repr__`. Less boilerplate than plain class.
**When:** Internal data structures you control. Config, results, briefing objects.
**Where:** Response objects, config holders — NOT tool inputs (use Pydantic for those).

```python
@dataclass
class Config:               # frozen=True → immutable (for settings)
    model: str
    temperature: float = 0.0

@dataclass
class TaskList:
    owner: str
    tasks: List[str] = field(default_factory=list)  # always use field() for lists
```

`__post_init__`: compute derived fields or validate after creation.

---

## P9 — `**kwargs`

**What:** Collect unknown keyword args into a dict.
**When:** Optional extras without defining every field. Forwarding args.
**Where:** Flexible tool wrappers, forwarding options to underlying APIs.
**Key rule:** Always last parameter. `kwargs.get("key")` for safe access.

```python
def send(to: str, body: str, **kwargs) -> str:
    urgent = kwargs.get("urgent", False)
    ...
# Unpack a dict: func(**my_dict) = func(key=val, ...)
```

---

## P10 — List Comprehensions

**What:** Build/filter lists in one line.
**When:** Transform or filter any collection.
**Where:** Processing tool results, filtering reminders, building summaries.

```python
names   = [c["name"] for c in contacts]                    # transform
active  = [c for c in contacts if c["active"]]              # filter
lines   = [f"[URGENT] {r['msg']}" if r["urgent"]
           else r["msg"] for r in pending]                  # conditional
book    = {c["name"]: c["phone"] for c in contacts}         # dict comp
```

`if m` skips falsy values (empty string, None, 0, []).

---

## P11 — Annotated Types

**What:** Attach extra rules to a type. Pydantic + LangGraph read and enforce them.
**When:** Reusable constrained types. LangGraph state reducers.
**Where:** BaseModel fields, TypedDict state fields, function params.

```python
# Reusable type — define once, use everywhere
def _valid_lang(v: str) -> str:
    if v not in ("english", "telugu"): raise ValueError(f"got '{v}'")
    return v
Language = Annotated[str, AfterValidator(_valid_lang)]
Seats    = Annotated[int, Field(ge=1, le=6)]

# LangGraph reducer — CRITICAL
class AgentState(TypedDict):
    messages: Annotated[list, add]   # nodes append, never overwrite
    actions:  Annotated[list, add]
```

Without `Annotated[list, add]` → each node overwrites history. Always use it for `messages`.

---

## P12 — Full LangChain Agent

**Four parts — always the same structure:**

```
STATE    → TypedDict + Annotated[list, add] for messages
TOOLS    → @tool + Pydantic + try/except
NODES    → agent_node (LLM), tool_node (ToolNode), should_continue (router)
GRAPH    → StateGraph → add_node → conditional_edges → compile → invoke
```

```python
# State
class AgentState(TypedDict):
    messages: Annotated[list, add]
    input: str; output: str; done: bool

# Tools
@tool("book_cab", args_schema=BookInput)
def book_cab(...) -> str:
    try: return ...
    except Exception as e: return f"Error: {e}"

# Model + bind
model = ChatAnthropic(model="claude-sonnet-4-6").bind_tools(tools)

# Nodes
def agent_node(state): return {"messages": [model.invoke(state["messages"])]}
tool_node = ToolNode(tools)
def router(state): return "tools" if state["messages"][-1].tool_calls else "end"

# Graph
g = StateGraph(AgentState)
g.add_node("agent", agent_node)
g.add_node("tools", tool_node)
g.set_entry_point("agent")
g.add_conditional_edges("agent", router, {"tools": "tools", "end": END})
g.add_edge("tools", "agent")
graph = g.compile()

# Run
result = graph.invoke({"input": "...", "messages": [], "output": "", "done": False})
```

---

## Quick Decision Table

| Situation | Use |
|-----------|-----|
| Agent remembers state between nodes | P3 TypedDict + P11 `Annotated[list, add]` |
| LLM fills in tool arguments | P4 Pydantic + P5 @tool |
| Multiple API calls at once | P6 asyncio.gather |
| Tool must never crash agent | P7 try/except → return string |
| Config that cannot change mid-run | P8 `@dataclass(frozen=True)` |
| Filter a list of reminders | P10 list comprehension |
| Optional extras in a function | P9 **kwargs |
| Constrained type used in many models | P11 Annotated + AfterValidator |
