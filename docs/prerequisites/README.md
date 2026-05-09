# Prerequisites

`deepagents` is built on top of two LangChain libraries:

- **`langchain`** (and `langchain-core`) — the LLM-and-tool abstraction layer.
  Provides `ChatModel`, message types, the `@tool` decorator, the `AgentMiddleware`
  base class, and the `create_agent()` factory that deepagents wraps.
- **`langgraph`** — the graph runtime that actually executes agents. Provides
  `StateGraph`, `MessagesState`, checkpointers, streaming modes, interrupts,
  and subgraphs.

You can read the deepagents docs without prior LangChain/LangGraph experience —
these two chapters give you exactly the API surface area you'll see used in
the codebase. They are deliberately scoped: we do not cover RAG, retrievers,
vector stores, or document loaders, none of which deepagents uses.

If you already know LangChain and LangGraph well, skim the chapters for the
deepagents-relevant call-outs and skip the rest:

| Chapter | Why deepagents needs it |
|---|---|
| [`langchain.md`](./langchain.md) | `init_chat_model`, messages, `@tool`, `bind_tools`, `create_agent`, **`AgentMiddleware`** (the hook system every deepagents middleware extends) |
| [`langgraph.md`](./langgraph.md) | `StateGraph`, `MessagesState`, reducers, the agent loop, **checkpointers** (sessions), **streaming modes** (the CLI's TUI), **interrupts** (HITL approvals), **subgraphs** (subagents) |

After reading these, the rest of the documentation can refer to "the agent
loop", "the `messages` channel", "a `wrap_model_call` hook", "the
checkpointer", or "an interrupt" without re-explaining each one.

## Sources

These chapters are distilled from the official LangChain and LangGraph docs at
<https://docs.langchain.com>. They are accurate as of LangChain 1.x / LangGraph
1.1+. If a deepagents file uses an API not covered here, it is either rare
enough to be explained inline in the file's doc, or the upstream docs have
changed and this chapter needs an update.
