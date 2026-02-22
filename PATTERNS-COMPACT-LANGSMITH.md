# LangSmith Patterns — Compact Reference
*8 patterns for observability, tracing, and evals. When/where to use each one.*
*Full reference: tool-langsmith.md skill | Last updated: 2026-02-22*

---

## LS1 — Setup

**What:** Configure environment variables so LangSmith receives traces.

**When:** Before running any application you want to observe. Required once per environment.

**Where:** `.env` file, shell profile, or CI secrets. Not in code.

**Key rule:** `LANGSMITH_TRACING=true` is mandatory — without it no traces are sent, even if
`@traceable` is applied. The env var is the single on/off switch.

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY=lsv2_...
export LANGSMITH_PROJECT=my-project   # "default" if unset; auto-created on first trace
```

---

## LS2 — @traceable Decorator

**What:** Wrap a Python function so every call becomes a traced run in LangSmith.

**When:** You control the function definition and want the simplest possible instrumentation.
Works for any function: chains, tools, retrievers, LLM calls.

**Where:** Around functions at the boundary between logical steps — pipeline entry points,
tools, retrieval functions.

**Key rule:** Nesting is automatic — a `@traceable` function called from inside another
`@traceable` function becomes a child span without any extra work.

```python
import langsmith as ls

@ls.traceable(run_type="tool", name="Retrieve Context", tags=["rag"])
def retrieve(query: str) -> str:
    return "retrieved context"

@ls.traceable
def pipeline(question: str) -> str:
    ctx = retrieve(question)   # automatically nested as child span
    return call_llm(question, ctx)
```

---

## LS3 — Manual Trace (context manager)

**What:** Trace a block of code without using a decorator. You explicitly set inputs and
outputs on the run tree object.

**When:** You cannot add a decorator (third-party code, lambdas, dynamic blocks), or you
need exact control over what is recorded as inputs/outputs.

**Where:** Around blocks that contain multiple steps you want to group under one span.

**Key rule:** Call `rt.end(outputs={...})` before the `with` block exits — otherwise outputs
are not recorded.

```python
import langsmith as ls

app_inputs = {"question": "Summarize the meeting"}

with ls.trace("Pipeline", "chain", project_name="my-project", inputs=app_inputs) as rt:
    result = run_my_pipeline(app_inputs["question"])
    rt.end(outputs={"answer": result})
```

---

## LS4 — Projects

**What:** Group traces into a named project (equivalent to an experiment or environment
namespace). The default project is `"default"`.

**When:** You want to separate dev / staging / prod traces, or group traces for a specific
experiment or version.

**Where:** Set once at startup via env var, or override dynamically per-call.

**Key rule:** Projects are created automatically on first trace — no manual setup in the UI
required.

```python
# Static (env var, applies to all traces in this process)
# LANGSMITH_PROJECT=experiment-v2

# Dynamic (overrides for one block)
import langsmith as ls
with ls.tracing_context(project_name="experiment-v2", enabled=True):
    pipeline("input")

# Per-call override
pipeline("input", langsmith_extra={"project_name": "experiment-v2"})
```

---

## LS5 — Run Metadata and Tags

**What:** Attach structured key-value metadata and string tags to any run (span).

**When:** You need to filter, group, or search traces in the LangSmith UI — by user, version,
environment, correlation ID, feature flag, etc.

**Where:** On the decorator, via `langsmith_extra` at call time, or dynamically on the run
tree inside the function body.

**Key rule:** Tags are `list[str]` — good for categorical labels. Metadata is `dict` — good
for structured values. Both are filterable in the UI.

```python
import langsmith as ls

@ls.traceable(tags=["prod", "rag-v2"], metadata={"model": "gpt-4.1-mini"})
def pipeline(question: str, user_id: str) -> str:
    rt = ls.get_current_run_tree()
    rt.metadata["user_id"] = user_id   # set dynamically inside function
    rt.tags.extend(["premium"])
    ...

# Or inject at call time without touching the function
pipeline("q", "u1", langsmith_extra={"metadata": {"ab_group": "B"}})
```

---

## LS6 — Datasets

**What:** A named, versioned collection of input/output example pairs used to run repeatable
evaluations.

**When:** You want to benchmark your application, track regression across versions, or build
a golden test set from production traces.

**Where:** Created once (SDK or UI), then referenced by name in evaluations.

**Key rule:** Datasets are made of `examples` — each example has `inputs` and optional
`outputs` (reference outputs). `outputs` become the ground truth that evaluators compare
against.

```python
from langsmith import Client

client = Client()
dataset = client.create_dataset("QA Gold Set", description="Golden QA pairs")
client.create_examples(
    dataset_id=dataset.id,
    examples=[
        {"inputs": {"question": "Capital of France?"}, "outputs": {"answer": "Paris"}},
        {"inputs": {"question": "Capital of Japan?"},  "outputs": {"answer": "Tokyo"}},
    ]
)
```

---

## LS7 — Evals (evaluate function)

**What:** Run your application over every example in a dataset, score each output with
evaluator functions, and record results as an experiment in LangSmith.

**When:** Pre-deployment regression testing, A/B comparison of prompt versions, benchmarking
model changes.

**Where:** In a dedicated eval script or CI pipeline step.

**Key rule:** Your target function receives `inputs: dict` and must return `dict`. Evaluators
receive `(run, example)` and return `{"key": ..., "score": ...}`. Use `aevaluate()` for large
datasets (async + `max_concurrency`).

```python
from langsmith.evaluation import evaluate

def my_app(inputs: dict) -> dict:
    return {"answer": call_llm(inputs["question"])}

def is_correct(run, example) -> dict:
    score = 1 if run.outputs["answer"] == example.outputs["answer"] else 0
    return {"key": "correctness", "score": score}

results = evaluate(
    my_app,
    data="QA Gold Set",          # dataset name
    evaluators=[is_correct],
    experiment_prefix="v2-test",
)
```

---

## LS8 — Prompt Hub

**What:** Version-controlled prompt storage. Push prompts to your workspace; pull them back
in code. Public prompts on the LangChain Hub are also pullable.

**When:** You want to iterate on prompts independently of code, share prompts across
services, or roll back to a previous prompt version.

**Where:** Prompt creation/management scripts; pulled at application startup or per-request.

**Key rule:** Pulling a private prompt needs only the prompt name. Pulling a public LangChain
Hub prompt requires the author handle prefix (`"handle/name"`).

```python
from langsmith import Client
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI

client = Client()

# Push (create or update)
prompt = ChatPromptTemplate.from_messages([("system", "You are helpful."), ("human", "{question}")])
client.push_prompt("my-qa-prompt", object=prompt)

# Pull private prompt
prompt = client.pull_prompt("my-qa-prompt")
chain = prompt | ChatOpenAI(model="gpt-4.1-mini")
chain.invoke({"question": "What is LangSmith?"})

# Pull public hub prompt (handle/name required)
prompt = client.pull_prompt("langchain-ai/sql-agent-system-prompt")

# Pull prompt + model as bundled runnable
chain = client.pull_prompt("my-qa-prompt-with-model", include_model=True)
```

---

## Quick Decision Table

| Situation | Use pattern |
|-----------|------------|
| I want to see all traces for my app | LS1 Setup + LS4 Projects |
| I own the function I want to trace | LS2 @traceable |
| I need to trace a block, not a function | LS3 Manual trace |
| I want to separate dev / prod traces | LS4 Projects |
| I need to filter traces by user or version | LS5 Metadata + Tags |
| I want repeatable tests against fixed inputs | LS6 Datasets |
| I want to score outputs and track over time | LS7 Evals |
| I want to version and share prompts | LS8 Prompt Hub |
| I need to A/B test two prompt versions | LS8 push (two versions) + LS7 evaluate both |
| Traces not appearing in UI | Check LS1: `LANGSMITH_TRACING=true` must be set |
