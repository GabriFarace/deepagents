# `libs/deepagents/deepagents/middleware/_tool_exclusion.py`

> Tool filtering middleware. It removes harness-excluded tools from each model
> request after other middleware has had a chance to inject tools.

## Position in the system

Profiles can mark tool names as excluded for a given model or harness. This
middleware is placed late in the stack so it can see the final tool list,
including tools added by filesystem, subagent, async subagent, skills, or other
middleware.

```
tool-injecting middleware
  └─ request.tools
       └─ _ToolExclusionMiddleware strips excluded names
            └─ model call
```

## Imports and module-level state

The module imports `AgentMiddleware` at runtime and keeps the rest of its type
imports under `TYPE_CHECKING`. There are no constants besides the constructor's
stored exclusion set.

## Functions and classes

### `_tool_name(tool)`

Extracts a name from either a LangChain `BaseTool` or a dict-shaped tool
schema. Dict tools must carry a string `"name"`; object tools use their `.name`
attribute. Anything without a string name returns `None`, which means it will
not match an exclusion entry.

### `_ToolExclusionMiddleware`

Stores a frozen set of tool names to remove. The class is intentionally private
because it is an assembly detail, not a user-facing middleware.

#### `wrap_model_call(request, handler)`

If exclusions are configured, builds a filtered `request.tools` list by keeping
only tools whose extracted name is not in `_excluded`. It then forwards the
overridden request to the next handler. It does not alter state, messages, or
model outputs.

#### `awrap_model_call(request, handler)`

Async equivalent of `wrap_model_call()`. It performs the same filtering before
awaiting the downstream model-call handler.

## Gotchas

The middleware removes tools only from the model request. It does not delete
middleware instances or source tool objects, so the same graph can still expose
different tools under a different profile or request configuration.
