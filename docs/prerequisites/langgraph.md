# LangGraph — what `deepagents` uses

> **Goal of this chapter.** Cover exactly the LangGraph surface area
> deepagents touches. We do not cover the functional API
> (`@entrypoint`, `@task`) since deepagents uses the imperative graph API.

---

## 1. The mental model

LangGraph models an agent (or any LLM workflow) as a **graph**:

- **State** — a typed dictionary that flows through the graph. Each node
  reads it and returns *partial updates* that get merged in via reducers.
- **Nodes** — Python functions that take the state and return updates.
- **Edges** — directed connections between nodes. Some are static; others
  are *conditional* (a function decides where to go next).

The **engine underneath is Pregel**: execution proceeds in discrete
"super-steps". In each super-step, every node that has new input runs in
parallel; their outputs are batched, applied to state via reducers, and the
next super-step picks up nodes that have new incoming messages. Execution
stops when no node has new input.

```
       START
         │
         ▼
       node_A ──► node_B ──► END
                    │
                    ▼  (conditional edge)
                  node_C ──► node_B   (cycles allowed)
```

For agents, the canonical shape is:

```
START → before_model → model → after_model → tools → before_model → ...
                                    │
                                    └── (no tool_calls) ──► END
```

---

## 2. State

### 2.1 Defining state

State can be a `TypedDict`, a `dataclass`, or a Pydantic `BaseModel`.
LangGraph (and deepagents) overwhelmingly use `TypedDict` because it's the
fastest and integrates with the merge-update protocol most naturally.

```python
from typing_extensions import TypedDict

class State(TypedDict):
    foo: int
    bar: list[str]
```

### 2.2 Reducers — how updates merge

When a node returns `{"foo": 2}`, LangGraph needs to know how to apply that
update to the existing state value. By default, the new value **overwrites**
the old. To customise, annotate the field:

```python
from typing import Annotated
from operator import add

class State(TypedDict):
    foo: int                            # default: overwrite
    bar: Annotated[list[str], add]      # reducer: list concat
```

If the input is `{"foo": 1, "bar": ["hi"]}` and a node returns
`{"bar": ["bye"]}`, the new state is `{"foo": 1, "bar": ["hi", "bye"]}`.

A reducer is a binary function `(current, update) -> new`. Common ones:

- `operator.add` — list concat
- `operator.or_` — set union, dict merge
- custom — last-writer-wins, max, etc.

### 2.3 `MessagesState` and `add_messages` — **the deepagents default**

For agents, you almost always want a `messages` channel that:

- appends new messages to the existing list,
- *but* if a new message has the same `id` as an existing one, replaces it
  (so HITL edits work),
- and deserialises dicts into `BaseMessage` objects automatically.

This is what `add_messages` does:

```python
from typing import Annotated
from typing_extensions import TypedDict
from langchain.messages import AnyMessage
from langgraph.graph.message import add_messages

class State(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
```

Or use the prebuilt:

```python
from langgraph.graph import MessagesState

class MyState(MessagesState):     # already has `messages` with add_messages
    documents: list[str]
```

deepagents' `AgentState` (from `langchain.agents.middleware`) extends
`MessagesState` with extra channels for todos, files, sub-agent state,
etc. Every middleware that needs a custom field declares it on a
`state_schema` subclass — see the LangChain primer §5.5.

---

## 3. Building a graph

### 3.1 `StateGraph`

```python
from langgraph.graph import StateGraph, START, END

builder = StateGraph(State)
builder.add_node("greet",  lambda s: {"foo": s["foo"] + 1})
builder.add_node("answer", lambda s: {"bar": s["bar"] + ["done"]})
builder.add_edge(START, "greet")
builder.add_edge("greet", "answer")
builder.add_edge("answer", END)

graph = builder.compile()
```

`StateGraph(State)` registers the state schema. `START` and `END` are
sentinel nodes — `START` is "where input enters", `END` is "where output
leaves". You **must** compile the graph before invoking it.

### 3.2 Nodes

A node is a function with one of these signatures:

```python
def node(state):                                    # most common
def node(state, config):                            # with RunnableConfig
def node(state, runtime: Runtime[ContextSchema]):   # with runtime context
async def node(state): ...                          # async
```

A node returns a `dict` (partial state update) or `None` (no update). If
not given an explicit name, the node takes the function's `__name__`.

`Runtime` (from `langgraph.runtime`) gives access to:

- `runtime.context` — typed runtime context (e.g., user_id, request_id)
- `runtime.store` — the long-term `BaseStore` (for cross-thread memory)
- `runtime.stream_writer` — for emitting custom stream events
- `runtime.execution_info` — thread/checkpoint/run identifiers

deepagents middlewares routinely use `runtime.store` and
`runtime.stream_writer`.

### 3.3 Edges

```python
builder.add_edge("a", "b")                                    # static
builder.add_conditional_edges("a", routing_fn)               # dynamic
builder.add_conditional_edges("a", routing_fn, {"yes": "b", "no": "c"})
```

`routing_fn(state)` returns the name of the next node, or a list of names
(parallel fan-out), or — with the path-map dict variant — a key that maps
to the next-node name. Returning `END` exits the graph.

### 3.4 `Command` — combine state update *and* routing

A node can return a `Command` instead of a dict to do both in one shot:

```python
from langgraph.types import Command

def my_node(state) -> Command:
    return Command(
        update={"messages": [AIMessage("done")]},
        goto="other_node",
    )
```

`Command(graph=Command.PARENT)` lets a subgraph navigate in its parent.
`Command(resume=...)` is used to resume after an interrupt (see §6).

### 3.5 `Send` — fan-out with per-target state

```python
from langgraph.types import Send

def fan_out(state):
    return [Send("worker", {"item": x}) for x in state["items"]]

builder.add_conditional_edges("dispatcher", fan_out)
```

Each `Send` runs the target node with a different sub-state. Results are
merged back into the parent state via reducers. deepagents uses this for
parallel sub-agent dispatch.

---

## 4. Compilation and invocation

### 4.1 Compile

```python
graph = builder.compile(
    checkpointer=...,        # for persistence (see §5)
    store=...,                # for cross-thread storage
    interrupt_before=["x"],   # static breakpoints
    interrupt_after=["y"],
    cache=...,                # node-level caching
)
```

### 4.2 Invocation methods

```python
graph.invoke(input, config={"configurable": {"thread_id": "abc"}})
graph.stream(input, config=..., stream_mode="updates", version="v2")

# async equivalents
await graph.ainvoke(input, config=...)
async for chunk in graph.astream(input, config=...):
    ...
```

`input` is a dict matching the state schema. `config` carries
`thread_id` (required for checkpointing), tags, callbacks, etc.

### 4.3 Calling subgraphs / introspection

```python
graph.get_graph().draw_ascii()       # ASCII visualisation
graph.get_state(config)              # current checkpoint state
graph.update_state(config, values)   # mutate state outside a node
graph.aget_state, graph.aupdate_state
```

These methods are how the deepagents CLI fetches `messages` history when
loading a session.

---

## 5. Persistence: checkpointers and threads

### 5.1 What a checkpointer does

After every super-step, if the graph was compiled with a checkpointer,
LangGraph saves the full state to durable storage. This enables:

- resuming a conversation later (sessions),
- pausing at an interrupt and resuming when the user replies,
- time-travel (jumping back to an earlier checkpoint),
- inspecting the trace.

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver
from langgraph.checkpoint.postgres import PostgresSaver

graph = builder.compile(checkpointer=MemorySaver())
```

deepagents-cli runs against `langgraph dev`, which uses an SQLite
checkpointer by default; in production deployments, Postgres is the norm.

### 5.2 Threads

A **thread** is a series of checkpoints sharing the same `thread_id`.
Reusing a `thread_id` resumes that thread; using a new one starts fresh.

```python
config = {"configurable": {"thread_id": "user-42-conv-1"}}
graph.invoke({"messages": [HumanMessage("hi")]}, config=config)
# ... later ...
graph.invoke({"messages": [HumanMessage("continue")]}, config=config)
# the second invocation sees the messages from the first
```

The deepagents CLI's "session" concept is exactly a thread plus a small
on-disk metadata file recording its title and timestamps.

### 5.3 Inspecting and editing thread state

```python
state = graph.get_state(config)
# state.values   -> full state dict
# state.next     -> tuple of next nodes (e.g. after an interrupt)
# state.config   -> includes the checkpoint id
# state.tasks    -> pending tasks
# state.created_at, state.metadata

graph.update_state(config, {"messages": [...]})  # forces a new checkpoint
```

`update_state` is how the CLI handles "edit a previous message and replay".

---

## 6. Interrupts — pause/resume for HITL

### 6.1 `interrupt()` inside a node

```python
from langgraph.types import interrupt

def approval_node(state):
    decision = interrupt({"action": "delete_file", "path": state["target"]})
    # ↑ when this fires, execution pauses; the payload is JSON-serialisable
    return {"approved": decision}
```

When `interrupt()` is called:

1. The graph's state is checkpointed.
2. The function's call stack is paused.
3. The graph's `.invoke()` / `.astream()` returns; the streamed payload
   includes the interrupt under `__interrupt__` / `chunk["interrupts"]`
   in v2.
4. The graph waits *forever* until you resume.

### 6.2 Resuming with `Command(resume=...)`

```python
# first call hits the interrupt
result = graph.invoke({"input": ...}, config=config, version="v2")
print(result.interrupts)  # the values you passed to interrupt()

# second call resumes
graph.invoke(Command(resume=True), config=config, version="v2")
```

The value you pass as `resume=` becomes the return value of the
`interrupt()` call inside the paused node. It can be any JSON-serialisable
type — bool, dict, etc.

This is **the foundation of every HITL approval flow** in deepagents (and
in Claude Code-style CLIs). The `HumanInTheLoopMiddleware` wraps risky tool
calls in `interrupt(...)` calls; the CLI presents an approval widget; the
user's choice is sent back as `Command(resume=...)`.

### 6.3 Static breakpoints

If you compiled with `interrupt_before=["tools"]`, the graph pauses *before*
the `tools` node every time, no `interrupt()` call needed. Less flexible
than dynamic interrupts but useful for debugging.

---

## 7. Streaming

LangGraph's streaming API is what powers the deepagents CLI's live TUI.

### 7.1 `stream()` / `astream()`

```python
for chunk in graph.stream(input, config=config, stream_mode="updates", version="v2"):
    ...
```

Always pass `version="v2"` (LangGraph ≥ 1.1) — it gives you a uniform
output shape regardless of stream modes used:

```python
{"type": "updates", "ns": (), "data": {...}}
```

### 7.2 Stream modes

| Mode | Payload | Use for |
|---|---|---|
| `values` | full state after each super-step | reactive UIs that re-render from full state |
| `updates` | per-node `{node_name: update}` after each super-step | logs, animations |
| `messages` | `(AIMessageChunk, metadata)` | **token-by-token streaming** |
| `custom` | whatever you wrote via `get_stream_writer()` | progress, status |
| `checkpoints` | checkpoint events | debugging |
| `tasks` | task start/finish events | debugging |
| `debug` | everything | full trace |

You can combine modes:

```python
for chunk in graph.stream(input, stream_mode=["updates", "messages"], version="v2"):
    if chunk["type"] == "updates":
        ...
    elif chunk["type"] == "messages":
        msg, metadata = chunk["data"]
        print(msg.content, end="", flush=True)
```

### 7.3 `messages` mode + filtering

`metadata` includes `langgraph_node`, `langgraph_step`, `tags`, etc. The
deepagents CLI uses these to know which node produced a chunk and to
suppress sub-agent token streams from the main view.

You can tag invocations to filter them:

```python
joke_model = init_chat_model("gpt-5-mini", tags=["joke"])
# later:
async for chunk in graph.astream(..., stream_mode="messages", version="v2"):
    if chunk["data"][1]["tags"] == ["joke"]:
        ...
```

The special tag `"nostream"` removes a model invocation from the
`messages` stream entirely (useful for internal structured-output calls).

### 7.4 `get_stream_writer()` — custom events from inside a node

```python
from langgraph.config import get_stream_writer

def my_node(state):
    writer = get_stream_writer()
    writer({"status": "thinking..."})
    ...
```

Yields a `{"type": "custom", "data": {"status": "thinking..."}}` chunk to
consumers using `stream_mode="custom"`. deepagents middlewares emit progress
events this way.

---

## 8. Subgraphs

A subgraph is a compiled `StateGraph` used as a node inside another graph.
Two ways to invoke:

```python
# 1. Direct: the subgraph reads/writes a *namespaced* slice of parent state
parent = StateGraph(ParentState).add_node("worker", subgraph)

# 2. With explicit translation: parent dispatches via a function
def dispatch(state):
    sub_input = {"x": state["foo"]}
    return Command(goto="next_after_sub", update={"...": subgraph.invoke(sub_input)})
```

deepagents' `task` tool uses subgraphs (one per sub-agent invocation) so
each sub-agent runs in its own isolated state and message list, and only
the result message bubbles up to the parent.

### Subgraphs and streaming

When `subgraphs=True`, stream events from the sub-graph come through with a
non-empty `ns` (namespace) tuple identifying which sub-graph emitted them.
The CLI uses this to route sub-agent output to a separate widget.

---

## 9. Long-term memory: `BaseStore`

Whereas a *checkpointer* persists state per thread, a `BaseStore`
persists data **across threads** — a key-value store keyed by
`(namespace, key)`:

```python
from langgraph.store.memory import InMemoryStore
from langgraph.store.postgres import PostgresStore

store = InMemoryStore()
graph = builder.compile(store=store)
```

Inside a node:

```python
def remember(state, runtime):
    runtime.store.put(("user", state["user_id"]), "preferences", {"theme": "dark"})

def recall(state, runtime):
    item = runtime.store.get(("user", state["user_id"]), "preferences")
    return {"theme": item.value["theme"]}
```

deepagents has a `StoreBackend` (a `BackendProtocol` implementation backed
by `BaseStore`) so an agent's filesystem can be persistent across threads.

---

## 10. Putting it all together — the canonical agent loop

Conceptually, what `create_agent()` builds is:

```
class AgentState(MessagesState):
    # plus middleware-contributed fields

builder = StateGraph(AgentState)

builder.add_node("model", call_model)        # invokes the bound chat model
builder.add_node("tools", run_tools)          # dispatches AIMessage.tool_calls

def should_continue(state):
    last = state["messages"][-1]
    return "tools" if last.tool_calls else END

builder.add_edge(START, "model")
builder.add_conditional_edges("model", should_continue)
builder.add_edge("tools", "model")           # loop back

agent = builder.compile(checkpointer=...)
```

Middleware adds extra wrapper nodes (`before_model`, `after_model`, etc.)
and conditional edges so node-style hooks can fire and so jumps work.

---

## 11. Cheat sheet

```python
from langgraph.graph import StateGraph, START, END, MessagesState
from langgraph.graph.message import add_messages
from langgraph.types import Command, Send, interrupt
from langgraph.runtime import Runtime
from langgraph.checkpoint.memory import MemorySaver
from langgraph.config import get_stream_writer

# State
class S(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    counter:  int

# Build
b = StateGraph(S)
b.add_node("a", lambda s: {"counter": s["counter"] + 1})
b.add_node("b", lambda s, runtime: {"counter": s["counter"] * 2})
b.add_edge(START, "a")
b.add_conditional_edges("a", lambda s: "b" if s["counter"] < 5 else END)
b.add_edge("b", "a")

graph = b.compile(checkpointer=MemorySaver())

# Run
config = {"configurable": {"thread_id": "t1"}}
graph.invoke({"messages": [], "counter": 0}, config=config)

# Stream
for chunk in graph.stream(input, config=config,
                          stream_mode=["updates", "messages"], version="v2"):
    ...

# Inspect / edit thread
state = graph.get_state(config)
graph.update_state(config, {"counter": 0})

# Interrupt + resume
def approve(state):
    decision = interrupt("Proceed?")
    return {"approved": decision}
# first call hits the interrupt; resume with:
graph.invoke(Command(resume=True), config=config, version="v2")
```

---

## 12. Where to next

You now have the full LangGraph surface deepagents uses. Next stop is the
deepagents core: `docs/libs/deepagents/deepagents/graph.md` documents
`create_deep_agent()` — which is, at heart, a curated `create_agent()` call
with a fixed middleware stack, a custom backend protocol, and a few quality-
of-life tools layered on top.
