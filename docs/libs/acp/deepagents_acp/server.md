# `acp/deepagents_acp/server.py`

> ACP bridge that translates Agent Client Protocol sessions, prompts, tool events, config changes, and permission interrupts into LangGraph deep-agent calls.

## Position in the system

This package sits at the boundary between deepagents and ACP clients. It imports the SDK graph/backends and ACP schema objects, then translates protocol calls into LangGraph invocations and session updates.

## Imports and module-level state

This file imports `__future__, json, dataclasses, typing, uuid, acp, acp.exceptions, acp.schema, deepagents, deepagents.backends, langgraph.checkpoint.memory, langgraph.graph.state` and other helpers.

## Functions and classes

### `AgentSessionContext`

Context for an agent session, including working directory, mode, and model. This class inherits from `object` and is the main object for this part of the module.

### `AgentServerACP`

ACP agent server that bridges Deep Agents with the Agent Client Protocol. This class inherits from `ACPAgent` and is the main object for this part of the module.

#### `AgentServerACP.__init__(self, agent: CompiledStateGraph | Callable[[AgentSessionContext], CompiledStateGraph], *, modes: SessionModeState | None=None, models: list[dict[str, str]] | None=None)`

Initialize the ACP agent server with the given agent factory or compiled graph. Args: agent: Either a compiled state graph or a factory function that creates one modes: Optional mode configuration (deprecated, use config_options instead) models: Optional list of available models with 'value', 'name', and optionally 'description' Key arguments are `agent`, `modes`, `models`. It mutates `self._cwd`, `self._agent_factory`, `self._agent`, `self._session_modes`, `self._session_mode_states`, `self._session_models`. Internally it delegates to `isinstance`, `super`, `ValueError`. It runs synchronously in the caller and returns directly.

#### `AgentServerACP.on_connect(self, conn: Client)`

Store the client connection for sending session updates. Key arguments are `conn`. It mutates `self._conn`. It runs synchronously in the caller and returns directly.

#### `AgentServerACP.initialize(self, protocol_version: int, client_capabilities: ClientCapabilities | None=None, client_info: Implementation | None=None, **kwargs: Any)`

Return server capabilities to the ACP client. Key arguments are `protocol_version`, `client_capabilities`, `client_info`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `InitializeResponse`, `AgentCapabilities`, `PromptCapabilities`. This is asynchronous and awaits I/O or framework operations before returning.

#### `AgentServerACP.new_session(self, cwd: str, mcp_servers: list[HttpMcpServer | SseMcpServer | McpServerStdio] | None=None, **kwargs: Any)`

Create a new agent session with the given working directory. Key arguments are `cwd`, `mcp_servers`. It mutates `self._session_cwds[...]`, `self._session_modes[...]`, `self._session_mode_states[...]`, `self._session_models[...]`. Internally it delegates to `NewSessionResponse`, `uuid4`, `len`. This is asynchronous and awaits I/O or framework operations before returning.

#### `AgentServerACP.set_session_mode(self, mode_id: str, session_id: str, **kwargs: Any)`

Switch the session to a different mode, resetting the agent. Key arguments are `mode_id`, `session_id`. It mutates `self._session_modes[...]`, `self._session_mode_states[...]`. Internally it delegates to `SetSessionModeResponse`, `SessionModeState`. This is asynchronous and awaits I/O or framework operations before returning.

#### `AgentServerACP.set_config_option(self, config_id: str, session_id: str, value: str | bool, **kwargs: Any)`

Update a configuration option for the session. Handles both mode and model switching. When switching models, the agent is reset to use the new model. Key arguments are `config_id`, `session_id`, `value`. It mutates `self._session_modes[...]`, `self._session_mode_states[...]`, `self._session_models[...]`. Internally it delegates to `SetSessionConfigOptionResponse`, `isinstance`, `RequestError`, `any`, `SessionModeState`, `type`. This is asynchronous and awaits I/O or framework operations before returning.

#### `AgentServerACP.cancel(self, session_id: str, **kwargs: Any)`

Cancel the current execution. Key arguments are `session_id`. It mutates `self._cancelled`. This is asynchronous and awaits I/O or framework operations before returning.

#### `AgentServerACP.prompt(self, prompt: list[TextContentBlock | ImageContentBlock | AudioContentBlock | ResourceContentBlock | EmbeddedResourceContentBlock], session_id: str, message_id: str | None=None, **kwargs: Any)`

Process a user prompt and stream the agent response. Key arguments are `prompt`, `session_id`, `message_id`. It mutates `self._cancelled`. Internally it delegates to `PromptResponse`, `RuntimeError`, `getattr`, `MemorySaver`, `isinstance`, `astream`. This is asynchronous and awaits I/O or framework operations before returning.

## Flow walk-through

1. `new_session()` creates an ACP session id, records the working directory, and initializes per-session mode/model selections. If modes or models were provided to the server factory, `_build_config_options()` exposes them as ACP `config_options` so clients can render selectors.
2. `set_session_mode()` and `set_config_option()` update those per-session selections and call the private reset path. When the server was constructed with a callable factory, reset rebuilds the LangGraph graph from `AgentSessionContext(cwd, mode, model)`; when it was constructed with a compiled graph, reset reuses that graph.
3. `prompt()` converts incoming ACP content blocks into LangChain multimodal content blocks, ensures the graph has a checkpointer, and streams `agent.astream(..., stream_mode=["messages", "updates"], subgraphs=True)` with `thread_id=session_id`.
4. Message stream chunks are split into user-visible text, partial tool-call chunks, and `ToolMessage` results. Tool-call chunks are accumulated until their JSON arguments parse; at that moment the ACP client receives a `start_tool_call()` or `start_edit_tool_call()` update with an ACP `ToolKind`.
5. Update stream chunks are watched for LangGraph interrupts and todo updates. HITL interrupts are converted into ACP permission prompts; `write_todos` updates become ACP plan entries.
6. Cancellation is cooperative. `cancel()` flips `_cancelled`; `prompt()` checks it between stream chunks and returns `PromptResponse(stop_reason="cancelled")` after resetting the flag for the next turn.

## Protocol-facing labels

The server exposes two config-option descriptions to ACP clients:

```text
Controls how the agent requests permission
```

```text
The LLM model to use for this session
```

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
