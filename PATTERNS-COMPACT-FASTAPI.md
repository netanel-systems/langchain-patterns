# FastAPI Patterns — Compact Reference
*8 patterns for building API interfaces for AI agents. When/where to use each one.*
*Full reference: web-fastapi.md skill | Last updated: 2026-02-22*

---

## FA1 — App setup

**What:** Create the FastAPI app instance with metadata. Run via uvicorn.
**When:** Every project. One app per service.
**Where:** `main.py` or `app/main.py` at project root.
**Key rule:** Build the graph / load models at startup — not inside endpoint handlers.

```python
from fastapi import FastAPI
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.graph = build_graph()   # once at startup
    yield

app = FastAPI(title="Agent API", version="1.0.0", lifespan=lifespan)
# uvicorn main:app --reload
```

---

## FA2 — POST endpoint

**What:** Accept JSON body, run logic, return JSON.
**When:** Every agent call. Anything that receives input and returns output.
**Where:** Agent runner, classify, summarise endpoints.
**Key rule:** Use `async def` when the handler awaits something (LLM, DB, HTTP).

```python
@app.post("/agent/", response_model=AgentResponse)
async def run_agent(request: AgentRequest):
    result = await app.state.graph.ainvoke(
        {"messages": [HumanMessage(content=request.message)]},
        config={"configurable": {"thread_id": request.session_id or "default"}}
    )
    return AgentResponse(reply=result["messages"][-1].content)
```

---

## FA3 — Pydantic request / response models

**What:** Typed, validated input/output shapes. Auto-generates OpenAPI docs.
**When:** Every endpoint that receives or returns structured data.
**Where:** Separate `models.py` or top of `main.py`.
**Key rule:** Use different models for input and output to filter sensitive fields.

```python
from pydantic import BaseModel, Field

class AgentRequest(BaseModel):
    message: str
    session_id: str | None = None
    max_tokens: int = Field(default=1000, ge=1, le=8000)

class AgentResponse(BaseModel):
    reply: str
    session_id: str

# response_model filters output — only fields declared in AgentResponse are returned
@app.post("/agent/", response_model=AgentResponse)
async def run_agent(request: AgentRequest): ...
```

---

## FA4 — Async endpoints

**What:** `async def` handlers that `await` coroutines. `def` handlers run in threadpool.
**When:** `async def` when calling LLMs, async DBs, httpx. `def` for blocking libs.
**Where:** All agent-facing endpoints.
**Key rule:** Never mix `await` into a `def` function — use `async def` or it will fail.

```python
# Use async def — LangChain/LangGraph all support await
@app.post("/agent/")
async def run_agent(request: AgentRequest):
    result = await graph.ainvoke({"messages": [HumanMessage(request.message)]})
    return {"reply": result["messages"][-1].content}

# Use def — blocking library, FastAPI runs it in threadpool automatically
@app.get("/health/")
def health():
    return {"status": "ok"}
```

---

## FA5 — CORS

**What:** Headers that tell browsers which origins can call your API.
**When:** Frontend (React, Vue, etc.) on a different domain/port than the API.
**Where:** Add once, at the top of `main.py`, before routes.
**Key rule:** Add CORS middleware before any other middleware. `allow_credentials=True` requires explicit origins — no `"*"`.

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://myapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
# For fully open public APIs: allow_origins=["*"], allow_credentials=False
```

---

## FA6 — Streaming response

**What:** Send data chunk-by-chunk over a long-lived HTTP connection. No response buffering.
**When:** LLM token streaming, file downloads, real-time logs.
**Where:** Any endpoint where the client should see partial results immediately.
**Key rule:** Use `media_type="text/event-stream"` for SSE (AI agents). Wrap generator in `try/except` — unhandled exceptions silently kill the stream.

```python
from fastapi.responses import StreamingResponse
import json

async def token_generator(message: str):
    try:
        async for event in graph.astream_events(
            {"messages": [HumanMessage(message)]}, version="v2"
        ):
            if event["event"] == "on_chat_model_stream":
                chunk = event["data"]["chunk"].content
                if chunk:
                    yield f"data: {json.dumps({'content': chunk})}\n\n"
        yield "data: [DONE]\n\n"
    except Exception as e:
        yield f"data: {json.dumps({'error': str(e)})}\n\n"

@app.post("/stream/")
async def stream_agent(request: AgentRequest):
    return StreamingResponse(
        token_generator(request.message),
        media_type="text/event-stream"
    )
```

---

## FA7 — Wrap LangGraph agent

**What:** Expose a compiled LangGraph graph as an HTTP endpoint.
**When:** Every agent deployment. This is the main pattern.
**Where:** `main.py` — build graph once, expose via POST.
**Key rule:** Build graph at startup (lifespan), not per-request. Pass `thread_id` for multi-turn memory.

```python
from langchain_core.messages import HumanMessage

# Sync invoke (no streaming)
@app.post("/agent/", response_model=AgentResponse)
async def run_agent(request: AgentRequest):
    graph = request.app.state.graph
    result = await graph.ainvoke(
        {"messages": [HumanMessage(content=request.message)]},
        config={"configurable": {"thread_id": request.session_id or "anon"}}
    )
    return AgentResponse(
        reply=result["messages"][-1].content,
        session_id=request.session_id or "anon"
    )

# Streaming (SSE)
@app.post("/agent/stream")
async def stream_agent(request: AgentRequest):
    graph = request.app.state.graph

    async def generate():
        async for event in graph.astream_events(
            {"messages": [HumanMessage(content=request.message)]},
            version="v2"
        ):
            if event["event"] == "on_chat_model_stream":
                chunk = event["data"]["chunk"].content
                if chunk:
                    yield f"data: {json.dumps({'content': chunk})}\n\n"
        yield "data: [DONE]\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

---

## FA8 — Background tasks

**What:** Run a function after the response is sent. Client doesn't wait.
**When:** Logging, analytics, sending emails, cache warm-up.
**Where:** Add `BackgroundTasks` as a parameter to any endpoint.
**Key rule:** For real job queues or heavy compute, use Celery + Redis. BackgroundTasks is for lightweight fire-and-forget only.

```python
from fastapi import BackgroundTasks

def log_interaction(session_id: str, message: str, reply: str):
    with open("interactions.log", "a") as f:
        f.write(f"{session_id} | {message} | {reply}\n")

@app.post("/agent/", response_model=AgentResponse)
async def run_agent(request: AgentRequest, background_tasks: BackgroundTasks):
    result = await app.state.graph.ainvoke(
        {"messages": [HumanMessage(content=request.message)]}
    )
    reply = result["messages"][-1].content
    background_tasks.add_task(log_interaction, request.session_id, request.message, reply)
    return AgentResponse(reply=reply, session_id=request.session_id or "anon")
```

---

## Quick decision table

| Situation | Pattern |
|-----------|---------|
| Need an HTTP API around an agent | FA1 + FA7 |
| Agent takes JSON input | FA3 Pydantic request model |
| Filter internal fields from output | FA3 separate response model + `response_model=` |
| Call LLM or async DB in handler | FA4 `async def` + `await` |
| Frontend on different port | FA5 CORS middleware |
| Real-time token streaming | FA6 StreamingResponse + SSE |
| Expose LangGraph graph as endpoint | FA7 wrap pattern |
| Log after response, don't block client | FA8 BackgroundTasks |
| Heavy compute / job queue | FA8 note: use Celery + Redis instead |
| Multi-turn agent sessions | FA7 + `thread_id` in config |
