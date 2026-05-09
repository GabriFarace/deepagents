# `repl/langchain_repl/interpreter.py`

> Parser, compiler, VM, and async executor for the small custom REPL language used by langchain-repl.

## Position in the system

This package provides an optional REPL tool middleware for LangChain/deepagents. It is independent of the main SDK and can be mounted as middleware when an agent needs compact programmable tool orchestration.

## Imports and module-level state

This file imports `__future__, asyncio, concurrent.futures, dataclasses, enum, typing, langchain_core.tools, langchain_core.tools.base`.
Module constants worth noticing: `_PRINT_SENTINEL`, `_PARALLEL_SENTINEL`.

## Functions and classes

### `Token`

A lexical token produced by the tokenizer. This class inherits from `object` and is the main object for this part of the module.

### `ParseError`

Raised when REPL source cannot be parsed. This class inherits from `ValueError` and is the main object for this part of the module.

### `ForeignObjectInterface`

Protocol for dispatching operations on foreign objects. Currently limited to sync invocation only. This class inherits from `Protocol` and is the main object for this part of the module.

#### `ForeignObjectInterface.supports(self, value: Any)`

Return whether this handler manages the provided runtime value. Key arguments are `value`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `ForeignObjectInterface.get_item(self, value: Any, key: Any)`

Resolve `value[key]` for a supported foreign object. Key arguments are `value`, `key`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `ForeignObjectInterface.resolve_member(self, value: Any, name: str)`

Resolve `value.name` for a supported foreign object. Key arguments are `value`, `name`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

#### `ForeignObjectInterface.call(self, value: Any, args: tuple[Any, ...])`

Invoke `value(*args)` for a supported foreign object. Key arguments are `value`, `args`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

### `Task`

Deferred callable execution specification for `parallel`. This class inherits from `object` and is the main object for this part of the module.

### `OpCode`

Opcode values for the interpreter virtual machine. This class inherits from `IntEnum` and is the main object for this part of the module.

### `Instruction`

This class inherits from `object` and is the main object for this part of the module.

### `ForLoopState`

This class inherits from `object` and is the main object for this part of the module.

### `VMState`

This class inherits from `object` and is the main object for this part of the module.

### `Interpreter`

Compile and evaluate the mini REPL language against a bound environment. This class inherits from `object` and is the main object for this part of the module.

#### `Interpreter.__init__(self, *, functions: Mapping[str, Callable[..., Any] | BaseTool] | None=None, state: MutableMapping[str, Any] | None=None, bindings: Mapping[str, Any] | None=None, foreign_interfaces: Sequence[ForeignObjectInterface]=(), runtime: ToolRuntime | None=None, max_concurrency: int | None=None)`

Initialize an interpreter with callable bindings and execution state. Key arguments are `functions`, `state`, `bindings`, `foreign_interfaces`, `runtime`, `max_concurrency`. It mutates `self._functions`, `self._bindings`, `self._foreign_interfaces`, `self._state`, `self._printed_lines`, `self._runtime`. Internally it delegates to `dict`, `tuple`. It runs synchronously in the caller and returns directly.

#### `Interpreter.env(self)`

Return a copy of the mutable interpreter state. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `dict`. It runs synchronously in the caller and returns directly.

#### `Interpreter.state(self)`

Return a copy of the mutable interpreter state. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `dict`. It runs synchronously in the caller and returns directly.

#### `Interpreter.bindings(self)`

Return the read-only external bindings available to programs. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `dict`. It runs synchronously in the caller and returns directly.

#### `Interpreter.printed_lines(self)`

Return captured lines produced by `print`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `list`. It runs synchronously in the caller and returns directly.

#### `Interpreter.compile(self, source: str)`

Compile source code into VM instructions. Key arguments are `source`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `compile`, `tokenize`. It runs synchronously in the caller and returns directly.

#### `Interpreter.evaluate(self, source: str, *, print_callback: Callable[[str], None] | None=None)`

Evaluate source synchronously and persist resulting state. Key arguments are `source`, `print_callback`. It mutates `self._state`. Internally it delegates to `compile`. It runs synchronously in the caller and returns directly.

#### `Interpreter.aevaluate(self, source: str, *, print_callback: Callable[[str], None] | None=None)`

Evaluate source asynchronously and persist resulting state. Key arguments are `source`, `print_callback`. It mutates `self._state`. Internally it delegates to `compile`. This is asynchronous and awaits I/O or framework operations before returning.

## Flow walk-through

1. `_Tokenizer` turns source into tokens with line/column metadata. It recognizes comments, strings, numbers, names, punctuation, comparison operators, and explicit `EOF`.
2. `_ProgramCompiler` parses the token stream and emits VM instructions. Structured forms such as `if ... then ... else ... end` and `for ... in ... do ... end` become jumps and loop state rather than Python control flow.
3. `Interpreter.compile()` is the public boundary for that parse/compile step. `evaluate()` and `aevaluate()` then execute the compiled program against a copy of the current interpreter state.
4. Foreign functions and `BaseTool` instances are normalized before invocation so injected LangChain runtime arguments do not leak into model-authored payloads. The async path can run deferred independent calls through `parallel([defer(...)])` with a bounded executor.
5. Successful execution persists the updated state and returns either printed output or the final expression value. Parse/runtime failures raise or are caught by the middleware layer, depending on which API is calling the interpreter.

## Gotchas

Several blocks are async and assume they run inside the owning framework event loop. Calling them from sync code needs the package CLI or an explicit async runner.
