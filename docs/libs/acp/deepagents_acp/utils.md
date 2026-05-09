# `acp/deepagents_acp/utils.py`

> Conversion and shell-command helper functions shared by the ACP server.

## Position in the system

This package sits at the boundary between deepagents and ACP clients. It imports the SDK graph/backends and ACP schema objects, then translates protocol calls into LangGraph invocations and session updates.

## Imports and module-level state

This file imports `__future__, re, shlex, typing`.
Module constants worth noticing: `DANGEROUS_SHELL_PATTERNS`, `_MAX_DISPLAY_COMMAND_LENGTH`.

## Functions and classes

### `convert_text_block_to_content_blocks(block: TextContentBlock)`

Convert an ACP text block to LangChain content blocks. Key arguments are `block`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

### `convert_image_block_to_content_blocks(block: ImageContentBlock)`

Convert an ACP image block to LangChain content blocks. Key arguments are `block`. It does not keep durable module state; effects come from returned values or delegated calls. It runs synchronously in the caller and returns directly.

### `convert_audio_block_to_content_blocks(block: AudioContentBlock)`

Convert an ACP audio block to LangChain content blocks. Raises: NotImplementedError: Audio content is not yet supported. Key arguments are `block`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `NotImplementedError`. It runs synchronously in the caller and returns directly.

### `convert_resource_block_to_content_blocks(block: ResourceContentBlock, *, root_dir: str)`

Convert an ACP resource block to LangChain content blocks. Key arguments are `block`, `root_dir`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `startswith`, `lstrip`, `len`. It runs synchronously in the caller and returns directly.

### `convert_embedded_resource_block_to_content_blocks(block: EmbeddedResourceContentBlock)`

Convert an ACP embedded resource block to LangChain content blocks. Raises: ValueError: If the block has neither a ``text`` nor ``blob`` property. Key arguments are `block`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `hasattr`, `ValueError`, `getattr`. It runs synchronously in the caller and returns directly.

### `contains_dangerous_patterns(command: str)`

Check if a command contains dangerous shell patterns. These patterns can be used to bypass allow-list validation by embedding arbitrary commands within seemingly safe commands. Args: command: The shell command to check. Returns: True if dangerous patterns are found, False otherwise. Key arguments are `command`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `any`, `search`, `bool`. It runs synchronously in the caller and returns directly.

### `extract_command_types(command: str)`

Extract all command types from a shell command, handling && separators. For security-sensitive commands (python, node, npm, uv, etc.), includes the full signature to avoid over-permissioning. Each sensitive command has a dedicated handler that extracts the appropriate signature. Signature extraction strategy: - python/python3: Include module name for -m, just flag for -c - node: Just flag for -e/-p (code execution) - npm/yarn/pnpm: Include subcommand, and script name for "run" - uv: Include subcommand, and tool name for "run" - npx: Include package name - Others: Just the base command Args: command: The full shell command string Returns: List of command signatures (base command + subcommand/module for sensitive commands) Examples: >>> extract_command_types("npm install") ['npm install'] >>> extract_command_types("cd /path && python -m pytest tests/") ['cd', 'python -m pytest'] >>> extract_command_types("python -m pip install package") ['python -m pip'] >>> extract_command_types("python -c 'print(1)'") ['python -c'] >>> extract_command_types("node -e 'console.log(1)'") ['node -e'] >>> extract_command_types("uv run pytest") ['uv run pytest'] >>> extract_command_types("npm run build") ['npm run build'] >>> extract_command_types("ls -la | grep foo") ['ls', 'grep'] >>> extract_command_types("cd dir && npm install && npm test") ['cd', 'npm install', 'npm test'] Key arguments are `command`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `split`, `strip`, `len`, `append`. It runs synchronously in the caller and returns directly.

### `truncate_execute_command_for_display(command: str)`

Truncate a command string to a maximum length for display. Key arguments are `command`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `len`. It runs synchronously in the caller and returns directly.

### `format_execute_result(command: str, result: str)`

Format execute tool result for better display. Args: command: The shell command that was executed result: The raw result string from the execute tool Returns: Formatted string with command, output, and exit code Key arguments are `command`, `result`. It does not keep durable module state; effects come from returned values or delegated calls. Internally it delegates to `split`, `rstrip`, `append`, `join`, `startswith`, `strip`. It runs synchronously in the caller and returns directly.

## Gotchas

Most behavior is delegated through framework protocols, so preserve method signatures when editing even if an argument appears unused locally.
