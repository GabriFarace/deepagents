# `deepagents_acp/__main__.py`

## High-Level Purpose

Entry point module that allows the `deepagents_acp` package to be executed as a module via `python -m deepagents_acp`. It launches a test ACP agent server using the asyncio event loop.

## Functions

### `main() -> None`

**Purpose:** Run the test ACP agent server by calling `asyncio.run(_serve_test_agent())`.

**Parameters:** None.

**Return Value:** None.

**Key Logic:** Delegates entirely to `_serve_test_agent()` from `deepagents_acp.server` wrapped in `asyncio.run()` for a synchronous entry point.

## Module-Level Behavior

When executed as `python -m deepagents_acp`, the `if __name__ == "__main__":` guard calls `main()`.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `asyncio` | stdlib | Provides `asyncio.run()` for running the async server |
| `_serve_test_agent` | `deepagents_acp.server` | The async function that starts the test ACP server |
