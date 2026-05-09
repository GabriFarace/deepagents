# `libs/deepagents/deepagents/middleware/memory.py`

> Agent memory middleware. It loads configured `AGENTS.md`-style files from a
> backend and injects their contents plus memory-writing guidelines into the
> system prompt.

## Position in the system

`MemoryMiddleware` gives a deep agent persistent, always-on context. Unlike
skills, memory is not discovered on demand; configured sources are loaded once
per agent state and then appended to every model request.

```
before_agent()
  └─ backend.download_files(sources)
       └─ state["memory_contents"]

wrap_model_call()
  └─ append <agent_memory> + <memory_guidelines> to system prompt
```

The middleware relies on a backend for file reads. If the same agent also has
filesystem tools, the prompt tells the model it may update memory with
`edit_file`, but this module itself does not expose any tools.

## Imports and module-level state

The module imports LangChain middleware types, `ToolRuntime` for resolving
backend factories, `ChatAnthropic` and content-block types for optional prompt
caching metadata, and `append_to_system_message()` for prompt injection.

`MEMORY_SYSTEM_PROMPT` is the model-visible prompt template. It wraps loaded
memory in `<agent_memory>` tags and then gives detailed instructions for when
to update memory.

## Functions and classes

### `MemoryState`

Extends `AgentState` with private `memory_contents`, a mapping of configured
source path to loaded text. Marking it private keeps raw memory contents out of
public final state while leaving them available to middleware hooks.

### `MemoryStateUpdate`

TypedDict returned by `before_agent()` and `abefore_agent()`. It contains the
new `memory_contents` mapping that LangGraph merges into state.

### `MemoryMiddleware`

Loads memory files from a backend and appends the formatted memory prompt to
each model request. The constructor stores a backend instance or backend
factory, an ordered list of source paths, and an Anthropic cache-control flag.

#### `_get_backend(state, runtime, config)`

Resolves the configured backend. Direct backend instances are returned as-is.
If the backend is callable, the method builds a synthetic `ToolRuntime` from
the current state/runtime/config and calls the factory. That lets memory share
the same backend factory pattern as tool middleware even though this hook is
not itself running inside a tool call.

#### `_format_agent_memory(contents)`

Renders the final prompt text. If no memory was loaded, or all configured
sources are empty/missing, it substitutes `(No memory loaded)`. Otherwise it
emits sections in `self.sources` order, each containing the source path and
content, then interpolates that body into `MEMORY_SYSTEM_PROMPT`.

#### `before_agent(state, runtime, config)`

Synchronously loads configured memory sources before the agent run. If
`memory_contents` is already present in state, it returns `None`, making memory
loading a once-per-session/checkpoint operation.

The method calls `backend.download_files()` for all sources. Missing files are
ignored; any other backend error raises `ValueError`. Successful byte content
is decoded as UTF-8 and stored by source path.

#### `abefore_agent(state, runtime, config)`

Async equivalent of `before_agent()`, using `backend.adownload_files()`. The
same caching, missing-file handling, UTF-8 decoding, and error behavior apply.

#### `modify_request(request)`

Reads `request.state["memory_contents"]`, formats the memory prompt, appends it
to the system message, and returns an overridden request. If
`add_cache_control=True` and the request model is exactly a `ChatAnthropic`
instance, it adds Anthropic `cache_control: {"type": "ephemeral"}` to the last
system content block.

That runtime model check matters because other middleware may override the
request model. Bedrock and Vertex wrappers are intentionally not treated as
`ChatAnthropic`.

#### `wrap_model_call(request, handler)` and `awrap_model_call(request, handler)`

Both hooks call `modify_request()` and then delegate to the downstream handler.
They do not inspect the model response or mutate state directly.

## System prompts and tool descriptions

`MEMORY_SYSTEM_PROMPT`:

```text
<agent_memory>
{agent_memory}
</agent_memory>

<memory_guidelines>
    The above <agent_memory> was loaded in from files in your filesystem. As you learn from your interactions with the user, you can save new knowledge by calling the `edit_file` tool.

    **Learning from feedback:**
    - One of your MAIN PRIORITIES is to learn from your interactions with the user. These learnings can be implicit or explicit. This means that in the future, you will remember this important information.
    - When you need to remember something, updating memory must be your FIRST, IMMEDIATE action - before responding to the user, before calling other tools, before doing anything else. Just update memory immediately.
    - When user says something is better/worse, capture WHY and encode it as a pattern.
    - Each correction is a chance to improve permanently - don't just fix the immediate issue, update your instructions.
    - A great opportunity to update your memories is when the user interrupts a tool call and provides feedback. You should update your memories immediately before revising the tool call.
    - Look for the underlying principle behind corrections, not just the specific mistake.
    - The user might not explicitly ask you to remember something, but if they provide information that is useful for future use, you should update your memories immediately.

    **Asking for information:**
    - If you lack context to perform an action (e.g. send a Slack DM, requires a user ID/email) you should explicitly ask the user for this information.
    - It is preferred for you to ask for information, don't assume anything that you do not know!
    - When the user provides information that is useful for future use, you should update your memories immediately.

    **When to update memories:**
    - When the user explicitly asks you to remember something (e.g., "remember my email", "save this preference")
    - When the user describes your role or how you should behave (e.g., "you are a web researcher", "always do X")
    - When the user gives feedback on your work - capture what was wrong and how to improve
    - When the user provides information required for tool use (e.g., slack channel ID, email addresses)
    - When the user provides context useful for future tasks, such as how to use tools, or which actions to take in a particular situation
    - When you discover new patterns or preferences (coding styles, conventions, workflows)

    **When to NOT update memories:**
    - When the information is temporary or transient (e.g., "I'm running late", "I'm on my phone right now")
    - When the information is a one-time task request (e.g., "Find me a recipe", "What's 25 * 4?")
    - When the information is a simple question that doesn't reveal lasting preferences (e.g., "What day is it?", "Can you explain X?")
    - When the information is an acknowledgment or small talk (e.g., "Sounds good!", "Hello", "Thanks for that")
    - When the information is stale or irrelevant in future conversations
    - Never store API keys, access tokens, passwords, or any other credentials in any file, memory, or system prompt.
    - If the user asks where to put API keys or provides an API key, do NOT echo or save it.

    **Examples:**
    Example 1 (remembering user information):
    User: Can you connect to my google account?
    Agent: Sure, I'll connect to your google account, what's your google account email?
    User: john@example.com
    Agent: Let me save this to my memory.
    Tool Call: edit_file(...) -> remembers that the user's google account email is john@example.com

    Example 2 (remembering implicit user preferences):
    User: Can you write me an example for creating a deep agent in LangChain?
    Agent: Sure, I'll write you an example for creating a deep agent in LangChain <example code in Python>
    User: Can you do this in JavaScript
    Agent: Let me save this to my memory.
    Tool Call: edit_file(...) -> remembers that the user prefers to get LangChain code examples in JavaScript
    Agent: Sure, here is the JavaScript example<example code in JavaScript>

    Example 3 (do not remember transient information):
    User: I'm going to play basketball tonight so I will be offline for a few hours.
    Agent: Okay I'll add a block to your calendar.
    Tool Call: create_calendar_event(...) -> just calls a tool, does not commit anything to memory, as it is transient information
</memory_guidelines>
```

This module exposes no tools. The prompt references `edit_file` because the
default deep agent usually also installs `FilesystemMiddleware`.

## Gotchas

Memory files are loaded only if `memory_contents` is absent from state. If the
underlying files change during a checkpointed conversation, this middleware
will not re-read them unless state is cleared or migrated.

The prompt-caching option is Anthropic-specific and follows the request model
at runtime, not the constructor's original model choice.
