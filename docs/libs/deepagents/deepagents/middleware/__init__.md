# `libs/deepagents/deepagents/middleware/__init__.py`

> Middleware package facade. It explains why Deep Agents uses LangChain
> middleware and re-exports the public middleware classes.

## Position in the system

`deepagents.middleware` is the import surface used by SDK users and by
`deepagents.__init__`. `graph.py:create_deep_agent()` imports concrete
middleware modules directly for default assembly, while external users usually
import from this package root.

```
deepagents.middleware
  ├─ re-exports filesystem, memory, skills, subagents, async subagents
  └─ re-exports summarization middleware and factories
```

The long module docstring is also architectural documentation: SDK middleware
can inject tools, edit model requests, mutate persistent state, and append
system instructions before every LLM call. Plain user-provided tools cannot do
that because they run only after the model asks for them.

## Imports and module-level state

The module imports only public classes and factories from sibling middleware
modules, then pins the exported names in `__all__`. There is no runtime logic,
no env-var read, and no state mutation at import time.

## Functions and classes

### Public re-exports

The file re-exports `AsyncSubAgent`, `AsyncSubAgentMiddleware`,
`FilesystemMiddleware`, `FilesystemPermission`, `MemoryMiddleware`,
`SkillsMiddleware`, `CompiledSubAgent`, `SubAgent`, `SubAgentMiddleware`,
`SummarizationMiddleware`, `SummarizationToolMiddleware`, and
`create_summarization_tool_middleware`.

These names are the stable package-root API for middleware consumers. Adding a
new middleware class to this package does not automatically expose it here; the
module import and `__all__` entry must both be updated.

## Gotchas

This file is intentionally not the full middleware registry. Internal helpers
such as `_ToolExclusionMiddleware` and `PatchToolCallsMiddleware` are not
exported here, even though they live in the same package.
