# LangChain — what `deepagents` uses

> **Goal of this chapter.** Give you exactly enough LangChain to read the
> deepagents source without having to leave the docs. We cover models,
> messages, tools, the prebuilt `create_agent()` factory, and the middleware
> system that **every deepagents middleware extends**.

---

## 1. The mental model

LangChain provides a **provider-agnostic interface to chat models** and a small
set of higher-level building blocks that compose into agents:

```
┌─────────────────────────────────────────────────────────────────┐
│ create_agent(model, tools, system_prompt, middleware=[…])       │
│   ↓                                                              │
│ produces a LangGraph StateGraph that runs the standard agent loop│
└─────────────────────────────────────────────────────────────────┘

Building blocks under the hood:

  ChatModel  ←→  Messages  (System / Human / AI / Tool)
      ↑                       ↑
   bind_tools(…)       tool_calls field on AIMessage
      ↑
  Tools  (BaseTool, @tool, StructuredTool)

Cross-cutting:
  AgentMiddleware  — hooks before_model / after_model / wrap_model_call /
                     wrap_tool_call / before_agent / after_agent
```

`deepagents` wraps `create_agent()` and ships a stack of `AgentMiddleware`
subclasses that inject filesystem tools, planning, summarisation, sub-agent
delegation, HITL, and so on. So the most important section of this chapter is
**§5 — Middleware**.

---

## 2. Chat models

### 2.1 `init_chat_model` — the universal constructor

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-5-nano")
# ↑ inferred provider = "openai" because of the model id prefix

model = init_chat_model("anthropic:claude-sonnet-4-6", temperature=0.1)
# ↑ explicit provider via "provider:model_id" syntax
```

`init_chat_model(model_string, **kwargs)` returns a `BaseChatModel` instance.
The provider is inferred from the model id, or you can prefix it
(`"anthropic:..."`, `"openai:..."`, `"google_genai:..."`,
`"openrouter:..."`, `"bedrock_converse:..."`, etc.). Common kwargs:

- `temperature: float`
- `max_tokens: int`
- `timeout: int` (seconds)
- `model_provider: str` (overrides inference)
- `output_version: "v0" | "v1"` (controls how content blocks are stored —
  see §3.3)

You can also instantiate provider classes directly:

```python
from langchain_anthropic import ChatAnthropic
from langchain_openai import ChatOpenAI

model = ChatAnthropic(model_name="claude-sonnet-4-6")
model = ChatOpenAI(model="gpt-5-nano", temperature=0.1)
```

deepagents resolves user-supplied model strings via `init_chat_model` (with a
table of provider-specific defaults) — see `libs/deepagents/deepagents/_models.py`.

### 2.2 Standard methods

Every `BaseChatModel` implements:

| Method | Signature | Returns |
|---|---|---|
| `invoke(input)` | sync | `AIMessage` |
| `stream(input)` | sync | iterator of `AIMessageChunk` |
| `batch(inputs)` | sync | `list[AIMessage]` |
| `ainvoke(input)` | async | `AIMessage` |
| `astream(input)` | async | async iterator of `AIMessageChunk` |
| `abatch(inputs)` | async | `list[AIMessage]` |

`input` may be a string (treated as a single `HumanMessage`), a list of
messages (`SystemMessage`/`HumanMessage`/`AIMessage`/`ToolMessage`), or a list
of OpenAI-style dicts (`{"role": "...", "content": "..."}`).

```python
response = model.invoke("Hello")            # str shortcut
response = model.invoke([HumanMessage("Hi")])
response = model.invoke([
    {"role": "system", "content": "You are terse."},
    {"role": "user",   "content": "Define entropy."},
])
```

### 2.3 Streaming

`.stream()` yields `AIMessageChunk` objects that **add together** into the
full message:

```python
full = None
for chunk in model.stream("Hi"):
    full = chunk if full is None else full + chunk
    print(chunk.text, end="", flush=True)
```

This is the foundation that LangGraph's `stream_mode="messages"` (the one the
deepagents CLI uses for token-by-token rendering) builds on.

### 2.4 `bind_tools` — declaring callable tools to the model

```python
def get_weather(location: str) -> str:
    """Get the current weather at a location."""
    ...

model_with_tools = model.bind_tools([get_weather])
response = model_with_tools.invoke("What's the weather in Paris?")
```

`bind_tools` accepts:

- Plain Python functions (signature is read; docstring becomes the tool
  description; types become the JSON schema).
- LangChain `BaseTool` instances (most flexible — see §4).
- Pydantic `BaseModel` classes (used as tool argument schemas).
- Provider-native dicts.

The returned object is the same chat model with tools bound; it emits
`tool_calls` on `AIMessage` (see §3.2).

### 2.5 `with_structured_output` — schema-constrained generation

```python
from pydantic import BaseModel

class Answer(BaseModel):
    summary: str
    confidence: float

structured_model = model.with_structured_output(Answer)
result: Answer = structured_model.invoke("Summarise quantum tunnelling.")
```

Used by some deepagents middlewares for parsing model output into a typed
schema. Internally, it either uses native structured-output APIs (OpenAI
function-calling, Anthropic tool-use) or JSON-mode with a parser, depending
on the provider.

---

## 3. Messages

Messages are the universal currency: they're what models accept and return,
and what LangGraph stores in the `messages` channel of the agent state.

### 3.1 The four core types

```python
from langchain.messages import (
    SystemMessage, HumanMessage, AIMessage, ToolMessage,
)
```

| Class | Role | Notes |
|---|---|---|
| `SystemMessage` | initial instructions | "You are a helpful assistant." |
| `HumanMessage` | user input | text or multimodal blocks |
| `AIMessage` | model output | carries `tool_calls`, `usage_metadata`, `response_metadata` |
| `ToolMessage` | result of a tool call | must carry `tool_call_id` matching the corresponding `AIMessage.tool_calls[i]["id"]` |

### 3.2 `AIMessage` — the heart of an agent loop

The fields you'll see referenced throughout deepagents:

| Attribute | Type | What it is |
|---|---|---|
| `.content` | `str \| list[dict]` | raw content (string for simple text replies; list of provider-native blocks for multimodal/reasoning/tool use) |
| `.text` | `str` | shortcut: text portion only |
| `.content_blocks` | `list[ContentBlock]` | normalised cross-provider view of `.content` (text / reasoning / tool_use / image / etc.) |
| `.tool_calls` | `list[dict]` | structured tool-call requests; **empty when the model finishes** |
| `.id` | `str` | stable identifier (LangChain-generated or provider-supplied) |
| `.usage_metadata` | `{"input_tokens", "output_tokens", "total_tokens", ...}` | token counts |
| `.response_metadata` | `dict` | raw provider response metadata |

A tool call looks like this:

```python
response.tool_calls
# [
#   {"name": "get_weather",
#    "args": {"location": "Paris"},
#    "id":   "call_abc"}
# ]
```

The agent loop's job (see §6) is: while `response.tool_calls` is non-empty,
execute each tool and append a corresponding `ToolMessage` (with matching
`tool_call_id`); then call the model again.

### 3.3 Content blocks (multimodal, reasoning, server-side tools)

Provider responses increasingly contain mixed content: thinking blocks,
citations, partial tool-use blocks, images. LangChain stores them in
`.content` in **provider-native form** but exposes a normalised
`.content_blocks` view:

```python
# Anthropic native
AIMessage(content=[
    {"type": "thinking", "thinking": "...", "signature": "WaUjzkyp..."},
    {"type": "text",     "text": "..."},
])

# Same message via .content_blocks (cross-provider view)
[{"type": "reasoning", "reasoning": "...", "extras": {"signature": "WaUjzkyp..."}},
 {"type": "text",      "text": "..."}]
```

Setting `output_version="v1"` on the model (or env var `LC_OUTPUT_VERSION=v1`)
makes `.content` itself store the normalised blocks, which is useful for
serialisation. deepagents widgets use `content_blocks` to render thinking
sections separately from text.

### 3.4 `ToolMessage`

```python
ToolMessage(
    content="Sunny, 72°F",            # what the model sees
    tool_call_id="call_abc",          # MUST match the AIMessage tool_call id
    name="get_weather",               # optional but helpful
    artifact={"raw": {...}},          # NOT sent to the model (downstream-only)
)
```

The `artifact` field stores supplementary data — raw API responses, debug
info, file blobs — that is available to the surrounding application but never
shipped to the LLM. deepagents uses `artifact` to attach diff payloads, file
contents, and HITL metadata to tool results.

---

## 4. Tools

### 4.1 The `@tool` decorator

```python
from langchain.tools import tool

@tool
def search(query: str) -> str:
    """Search the web for `query` and return a short result string."""
    return f"Results for: {query}"
```

The decorator wraps a plain function into a `StructuredTool`. The signature
becomes the JSON schema; the docstring becomes the description (this is what
the model sees, so it should be precise). Decorator options worth knowing:

```python
@tool("custom_name", description="Override the docstring", return_direct=False)
def f(x: int) -> str: ...

@tool(args_schema=MyPydanticModel)            # explicit schema
def g(): ...

@tool(parse_docstring=True)                   # extract per-arg descriptions from docstring
def h(x: int, y: int) -> int:
    """Add two numbers.

    Args:
        x: the first number
        y: the second number
    """
```

### 4.2 `BaseTool` and `StructuredTool` (class-based)

For more control:

```python
from langchain.tools import BaseTool
from pydantic import BaseModel

class WeatherInput(BaseModel):
    location: str

class WeatherTool(BaseTool):
    name: str = "get_weather"
    description: str = "Get the current weather at a location."
    args_schema: type = WeatherInput

    def _run(self, location: str) -> str: ...
    async def _arun(self, location: str) -> str: ...
```

deepagents tools (read/write/edit/grep/glob/ls/execute) are implemented as
`BaseTool` subclasses to share state-injection plumbing.

### 4.3 The tool call lifecycle

1. **Bind** — `model.bind_tools([t1, t2, ...])` registers tools with the model.
2. **Call** — model emits `AIMessage(tool_calls=[{name, args, id}, ...])`.
3. **Dispatch** — your code (or the agent loop) executes each tool.
4. **Reply** — append a `ToolMessage(content=..., tool_call_id=...)` per call.
5. **Continue** — invoke the model again with the updated message history.

The model may emit multiple tool calls in parallel in a single `AIMessage`.
Dispatching them concurrently is the agent runtime's responsibility.

### 4.4 `tool_choice`

Some models support forcing a tool to be called:

```python
model.bind_tools(tools, tool_choice="get_weather")  # force this tool
model.bind_tools(tools, tool_choice="any")           # force any tool
model.bind_tools(tools, tool_choice="auto")          # let the model decide (default)
```

---

## 5. The middleware system — **the most important part**

This is the foundation `deepagents` builds on. Every deepagents feature
(filesystem tools, the `task` sub-agent, planning, summarisation, HITL
approval, prompt caching) is implemented as an `AgentMiddleware`.

### 5.1 What middleware is for

> Middleware lets you intercept every step of `create_agent()`'s execution
> loop and either observe, mutate, short-circuit, or retry the call.

Use cases:

- Logging and analytics
- Prompt mutation (system-prompt assembly, dynamic prompts)
- Tool-list filtering or runtime tool registration
- Retries, fallbacks, rate limits
- Early termination
- HITL approval gates
- Context compaction
- PII / guardrails

### 5.2 The hook menu

```
┌─────────────────────── one invocation ──────────────────────┐
│ before_agent  ─→  before_model  ─→  wrap_model_call  ─→ MODEL│
│                                          │                    │
│                                          ↓                    │
│ after_agent  ←─  after_model  ←─  wrap_tool_call  ←──  TOOLS  │
└─────────────────────────────────────────────────────────────┘
```

| Hook | Style | Fires |
|---|---|---|
| `before_agent` | node | once per `.invoke()`, before any model call |
| `before_model` | node | before every model call |
| `wrap_model_call` | wrap | around every model call (you decide when to call `handler(request)`) |
| `wrap_tool_call` | wrap | around every tool call |
| `after_model` | node | after every model response |
| `after_agent` | node | once per `.invoke()`, after the final model response |

**Node-style hooks** return a `dict | None`. The dict is merged into the
agent state via the graph's reducers. They run **sequentially** — `before_*`
in registration order, `after_*` in reverse order.

**Wrap-style hooks** are middleware in the WSGI/Express sense: they receive
a `request` and a `handler`, they can call `handler(request)` zero, one, or
many times, and they return a response. They **nest**: the first middleware
wraps all others.

### 5.3 Class form (the canonical deepagents shape)

```python
from langchain.agents.middleware import (
    AgentMiddleware, AgentState, ModelRequest, ModelResponse,
)
from langgraph.runtime import Runtime

class LoggingMiddleware(AgentMiddleware):
    def before_model(self, state: AgentState, runtime: Runtime) -> dict | None:
        print(f"[{len(state['messages'])} messages] calling model")
        return None

    def after_model(self, state: AgentState, runtime: Runtime) -> dict | None:
        print("model returned:", state["messages"][-1].content[:80])
        return None

    def wrap_model_call(self, request, handler):
        # short-circuit, retry, cache, mutate request — all possible
        return handler(request)

    # async equivalents (called when the agent is invoked via .ainvoke)
    async def abefore_model(self, state, runtime): ...
    async def aafter_model(self, state, runtime): ...
    async def awrap_model_call(self, request, handler): ...
```

Pass to `create_agent(..., middleware=[LoggingMiddleware()])`.

### 5.4 Decorator form (for one-shot middlewares)

```python
from langchain.agents.middleware import (
    before_model, after_model, wrap_model_call,
)

@before_model
def log_before(state, runtime):
    print("before model")
    return None

@wrap_model_call
def retry_three_times(request, handler):
    for attempt in range(3):
        try:
            return handler(request)
        except Exception:
            if attempt == 2:
                raise
```

These produce middleware "shells" you can pass into `create_agent` directly.

### 5.5 Custom state schemas

A middleware can extend the agent state with new fields:

```python
from typing_extensions import NotRequired

class MyState(AgentState):
    model_call_count: NotRequired[int]
    user_id: NotRequired[str]

class CounterMiddleware(AgentMiddleware[MyState]):
    state_schema = MyState

    def before_model(self, state: MyState, runtime) -> dict | None:
        if state.get("model_call_count", 0) > 10:
            return {"jump_to": "end"}                   # early exit
        return None

    def after_model(self, state: MyState, runtime) -> dict | None:
        return {"model_call_count": state.get("model_call_count", 0) + 1}
```

`AgentState` already includes `messages: list[BaseMessage]` (with the
`add_messages` reducer). When you set `state_schema = MyState`, your fields
get merged into the graph's overall state schema. Multiple middlewares with
custom schemas compose: the union of their schemas is the final state shape.

### 5.6 Jumps — early termination from a node-style hook

Returning `{"jump_to": "end" | "model" | "tools"}` from a node-style hook
lets that hook skip parts of the loop:

```python
@before_model(can_jump_to=["end"])
def cap_messages(state, runtime):
    if len(state["messages"]) >= 50:
        return {
            "messages": [AIMessage("Conversation limit reached.")],
            "jump_to": "end",
        }
    return None
```

You must declare the legal jump targets via `can_jump_to=[...]` or the
`@hook_config(can_jump_to=...)` decorator, otherwise LangGraph won't wire
the conditional edges. Targets are: `"end"`, `"model"`, `"tools"`.

### 5.7 The `ModelRequest` / `ModelResponse` of `wrap_model_call`

```python
def wrap_model_call(self, request: ModelRequest, handler) -> ModelResponse:
    # request fields:
    request.state          # AgentState (read-only here)
    request.runtime        # Runtime (context, store)
    request.model          # BaseChatModel
    request.messages       # list[BaseMessage] (about to be sent)
    request.system_prompt  # str | None
    request.tools          # list[BaseTool]
    request.response_format
    request.tool_choice
    request.model_settings

    # mutate via .override(...)
    new_request = request.override(model=other_model, tools=fewer_tools)

    response = handler(new_request)
    return response   # AIMessage (or ExtendedModelResponse with a Command)
```

Returning `ExtendedModelResponse(model_response=response, command=Command(update={...}))`
lets you both produce the model's reply and inject custom state updates.

### 5.8 Composition order

Given `middleware=[A, B, C]`:

```
before_agent : A → B → C
loop start
  before_model : A → B → C
  wrap_model_call : A( B( C( model ) ) )      # nested; A is outermost
  after_model : C → B → A                      # reverse
  (maybe tools, then back to before_model)
loop end
after_agent : C → B → A                        # reverse
```

deepagents fixes the ordering of its built-in stack carefully — see
`docs/libs/deepagents/deepagents/middleware/README.md` for the canonical list
and rationale.

---

## 6. `create_agent` — the prebuilt agent loop

```python
from langchain.agents import create_agent

agent = create_agent(
    model="anthropic:claude-sonnet-4-6",
    tools=[get_weather, search],
    system_prompt="You are a helpful assistant.",
    middleware=[LoggingMiddleware()],
)

result = agent.invoke({
    "messages": [{"role": "user", "content": "What's the weather in Paris?"}]
})
```

Under the hood, `create_agent` returns a compiled LangGraph
`StateGraph`. The graph has (at minimum):

- a `model` node that calls the (tool-bound) chat model
- a `tools` node that executes tool calls
- conditional edges that loop `model → tools → model` while there are tool
  calls, and exit to `END` when the model returns no `tool_calls`.

Middleware adds extra nodes and edges around this skeleton.

The **input** to `agent.invoke` / `agent.astream` is a dict matching the
agent state schema; the only required key is `"messages"`. The **output** is
the final state — `result["messages"]` is the full conversation including
tool calls and tool messages. `result["messages"][-1]` is the model's last
reply.

### Static vs. dynamic model selection

```python
# static (most common)
agent = create_agent(model="anthropic:claude-sonnet-4-6", tools=...)

# dynamic via wrap_model_call middleware
@wrap_model_call
def pick_model(request, handler):
    if len(request.state["messages"]) > 10:
        request = request.override(model=advanced_model)
    return handler(request)
```

deepagents' profiles system (`libs/deepagents/deepagents/profiles/`) is
a more elaborate version of this: per-model adjustments to system prompt,
tool descriptions, and middleware are applied as wrap-style middleware.

---

## 7. Cheat sheet

```python
# Models
model = init_chat_model("anthropic:claude-sonnet-4-6", temperature=0)
model_with_tools = model.bind_tools([t1, t2])
structured = model.with_structured_output(Schema)

# Messages
[SystemMessage("..."), HumanMessage("..."), AIMessage("..."), ToolMessage("...", tool_call_id="...")]
ai.tool_calls       # [{name, args, id}, ...]
ai.usage_metadata   # {input_tokens, output_tokens, total_tokens}
ai.content_blocks   # normalised cross-provider view

# Tools
@tool
def f(x: int) -> str: """..."""

# Agents
agent = create_agent(model, tools=..., system_prompt="...", middleware=[...])
agent.invoke({"messages": [...]})
async for chunk in agent.astream({"messages": [...]}, stream_mode="messages"):
    ...

# Middleware
class M(AgentMiddleware):
    state_schema = MyState                        # optional
    def before_model(self, state, runtime): ...   # node hook
    def wrap_model_call(self, request, handler):  # wrap hook
        return handler(request.override(...))
    @hook_config(can_jump_to=["end"])
    def after_model(self, state, runtime): ...    # can early-exit
```

---

## 8. Where to next

You are now ready to read [`langgraph.md`](./langgraph.md), which covers the
graph runtime that actually executes everything described here.
