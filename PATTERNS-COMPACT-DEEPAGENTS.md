# DeepAgents Patterns — Compact Reference
*8 patterns for autonomous agents with planning, tools, memory, and subagents.*
*Full reference: agent-deepagents.md skill | Last updated: 2026-02-22*

---

## DA1 — create_deep_agent (core)

**What:** Single function creates a full autonomous agent. Built on LangGraph.
**When:** Any autonomous agent — coding, research, writing, task execution.
**Where:** Entry point of every DeepAgents project. One call = complete agent.
**Key rule:** DeepAgents is built ON LangGraph. Same invoke/ainvoke interface.

```python
from deepagents import create_deep_agent

# Minimal
agent = create_deep_agent(model="anthropic:claude-sonnet-4-5-20250929")

# Full config
agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-5-20250929",
    tools=[my_tool],
    middleware=[MyMiddleware()],
    backend=FilesystemBackend(root_dir="./"),
    memory=["./AGENTS.md"],
    skills=["./skills/"],
    checkpointer=InMemorySaver(),
)

result = agent.invoke({"messages": [{"role": "user", "content": "Build a todo app"}]})
print(result["messages"][-1].content)
```

---

## DA2 — Tools (built-in + custom)

**What:** Built-in tools for files, shell, todos. Add your own with `@tool`.
**When:** Agent needs to read/write files, run commands, or call external APIs.
**Where:** `tools=[...]` param on `create_deep_agent`. Middleware adds built-ins automatically.
**Key rule:** Built-ins come from middleware, not `tools=`. Add FilesystemMiddleware to get file tools.

```python
# Built-in tools (via middleware — see DA3)
# ls, read_file, write_file, edit_file, glob, grep, execute
# write_todos, read_todos
# task (subagent delegation)

# Custom tool
from langchain_core.tools import tool

@tool
def search_web(query: str) -> str:
    """Search the web for information."""
    return tavily_client.search(query)

agent = create_deep_agent(tools=[search_web])
```

| Built-in Tool | What it does | Comes from |
|---------------|-------------|------------|
| `ls` | List directory | FilesystemMiddleware |
| `read_file` | Read file (paginated) | FilesystemMiddleware |
| `write_file` | Create/overwrite file | FilesystemMiddleware |
| `edit_file` | String replacement | FilesystemMiddleware |
| `execute` | Shell commands | FilesystemMiddleware |
| `write_todos` | Create task list | TodoListMiddleware |
| `task` | Delegate to subagent | SubAgentMiddleware |

---

## DA3 — Middleware

**What:** Injects tools and modifies system prompts. The plugin system.
**When:** Need file access, task planning, or subagent delegation.
**Where:** `middleware=[...]` param. Order matters — applied in sequence.
**Key rule:** Middleware adds tools automatically. No need to list them in `tools=`.

```python
from deepagents.middleware import (
    FilesystemMiddleware,
    TodoListMiddleware,
    SubAgentMiddleware,
)

agent = create_deep_agent(
    middleware=[
        FilesystemMiddleware(),    # adds ls, read_file, write_file, edit_file, execute
        TodoListMiddleware(),      # adds write_todos, read_todos
        SubAgentMiddleware(),      # adds task (delegate to subagent)
    ]
)
```

Custom middleware:
```python
from deepagents import AgentMiddleware

class MyMiddleware(AgentMiddleware):
    def get_tools(self): return [my_custom_tool]
    def get_system_prompt(self): return "Always respond in bullet points."
```

---

## DA4 — Backends (where files live)

**What:** Controls where the agent stores and reads files.
**When:** Any agent that works with files or needs persistence.
**Where:** `backend=` param. Default is StateBackend (ephemeral — lost on restart).
**Key rule:** StateBackend = dev only. FilesystemBackend = real files. CompositeBackend = both.

```python
from deepagents.backends import (
    StateBackend,        # ephemeral — files in memory only
    FilesystemBackend,   # real filesystem
    CompositeBackend,    # route by path prefix
    StoreBackend,        # LangGraph store (long-term memory)
)

# Real files (production)
agent = create_deep_agent(
    backend=FilesystemBackend(root_dir="./workspace")
)

# Route: memories → store, everything else → filesystem
from langgraph.store.memory import InMemoryStore
agent = create_deep_agent(
    backend=CompositeBackend(
        default=FilesystemBackend(root_dir="./"),
        routes={"/memories/": StoreBackend(store=InMemoryStore())}
    )
)
```

---

## DA5 — Memory + Skills (AGENTS.md)

**What:** Persistent personality + on-demand workflows loaded from markdown.
**When:** Agent needs consistent behavior across sessions, or specialized workflows.
**Where:** `memory=["./AGENTS.md"]` for persistent facts. `skills=["./skills/"]` for workflows.
**Key rule:** AGENTS.md is always loaded. Skills are loaded on demand when relevant.

```python
agent = create_deep_agent(
    memory=["./AGENTS.md"],      # always loaded — personality, preferences
    skills=["./skills/"],         # loaded on demand — specialized workflows
)
```

**AGENTS.md format:**
```markdown
# Agent Identity
You are Aria, a personal assistant for Klement.
Always respond in a friendly, concise tone.
Prefer Telugu when Klement seems stressed.
```

**Skills format** (`./skills/web-research/SKILL.md`):
```markdown
---
name: web-research
description: Research topics using web search
---
# Web Research Skill
Steps: 1. Search. 2. Summarize. 3. Cite sources.
```

---

## DA6 — Subagents (delegation)

**What:** Orchestrator delegates tasks to specialized sub-agents.
**When:** Complex tasks needing specialization — research + write + review = 3 subagents.
**Where:** `subagents=[...]` param. Orchestrator uses `task` tool to delegate.
**Key rule:** Each subagent has isolated context. Pass all needed info in the description.

```python
agent = create_deep_agent(
    subagents=[
        {
            "name": "researcher",
            "description": "Research topics before writing",
            "system_prompt": "You are a research specialist. Be thorough.",
            "tools": [search_web],
            "model": "anthropic:claude-haiku-4-5-20251001",  # cheaper model for subagent
        },
        {
            "name": "writer",
            "description": "Write polished content from research",
            "system_prompt": "You are a professional writer.",
            "model": "anthropic:claude-sonnet-4-5-20250929",
        },
    ]
)
# Orchestrator calls: task(subagent_type="researcher", description="Research X")
```

---

## DA7 — HITL (Human in the Loop)

**What:** Pause agent and ask human to approve/reject/edit before dangerous actions.
**When:** Agent writes files, runs shell commands, or makes irreversible changes.
**Where:** `interrupt_on={...}` + `checkpointer=InMemorySaver()` (required for HITL).
**Key rule:** No checkpointer = no HITL. Always pair them.

```python
from langgraph.checkpoint.memory import InMemorySaver

agent = create_deep_agent(
    checkpointer=InMemorySaver(),   # REQUIRED for HITL
    interrupt_on={
        "write_file": {"allowed_decisions": ["approve", "reject"]},
        "execute":    {"allowed_decisions": ["approve", "reject", "edit"]},
    }
)

# Resume after human decision
result = agent.invoke(input, config)
# → pauses at write_file, waits for human
agent.invoke(Command(resume="approve"), config)  # resume
```

---

## DA8 — Async + Streaming

**What:** Run agent asynchronously, stream tokens as they arrive.
**When:** Web APIs, real-time UIs, long-running tasks.
**Where:** Replace `invoke` → `ainvoke`. Stream with `astream`.
**Key rule:** Same pattern as LangGraph — DeepAgents inherits all LangGraph streaming modes.

```python
# Async invoke
result = await agent.ainvoke(
    {"messages": [{"role": "user", "content": "Build a todo app"}]}
)

# Stream tokens (messages mode)
async for chunk, metadata in agent.astream(
    {"messages": [{"role": "user", "content": "Research LangGraph"}]},
    stream_mode="messages"
):
    if hasattr(chunk, "content") and chunk.content:
        print(chunk.content, end="", flush=True)
```

---

## Quick Decision Table

| Situation | Use |
|-----------|-----|
| Start any autonomous agent | DA1 `create_deep_agent()` |
| Need file read/write/shell | DA3 FilesystemMiddleware |
| Agent plans its own tasks | DA3 TodoListMiddleware |
| Agent needs real file persistence | DA4 FilesystemBackend |
| Agent remembers its personality | DA5 AGENTS.md |
| Complex task → specialize | DA6 subagents |
| Dangerous actions need approval | DA7 HITL + interrupt_on |
| Real-time streaming output | DA8 astream |
| Multi-turn memory | DA1 checkpointer + thread_id |
