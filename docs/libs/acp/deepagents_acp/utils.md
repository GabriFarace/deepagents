# `deepagents_acp/utils.py`

## High-Level Purpose

Utility functions for converting ACP (Agent Client Protocol) content blocks into the dict-based format expected by LangChain message content, and for parsing and formatting shell command strings. These utilities are used internally by `AgentServerACP` to bridge the ACP protocol types with LangChain's internal representations.

## Functions

### `convert_text_block_to_content_blocks(block: TextContentBlock) -> list[dict[str, str]]`

**Purpose:** Convert an ACP `TextContentBlock` into a list of LangChain-style content dicts.

**Parameters:**
- `block`: An ACP `TextContentBlock` with a `.text` attribute.

**Return Value:** A single-element list: `[{"type": "text", "text": block.text}]`.

---

### `convert_image_block_to_content_blocks(block: ImageContentBlock) -> list[dict[str, object]]`

**Purpose:** Convert an ACP `ImageContentBlock` to a LangChain `image_url` content block.

**Parameters:**
- `block`: An ACP `ImageContentBlock` with `.data` (base64 string) and `.mime_type`.

**Return Value:**
- If `block.data` is truthy: `[{"type": "image_url", "image_url": {"url": "data:<mime>;base64,<data>"}}]`
- Otherwise: `[{"type": "text", "text": "[Image: no data available]"}]`

**Key Logic:** Constructs a data URI from the inline base64 image data.

---

### `convert_audio_block_to_content_blocks(block: AudioContentBlock) -> list[dict[str, str]]`

**Purpose:** Placeholder for audio block conversion. Audio is not currently supported.

**Raises:** `NotImplementedError` always.

---

### `convert_resource_block_to_content_blocks(block: ResourceContentBlock, *, root_dir: str) -> list[dict[str, str]]`

**Purpose:** Convert an ACP `ResourceContentBlock` (a file-link reference) into a readable text representation.

**Parameters:**
- `block`: ACP resource block with `.name`, `.uri`, `.description`, `.mime_type`.
- `root_dir`: The agent's working directory; stripped from URIs to produce relative paths.

**Return Value:** A single-element list with a `text` block that formats the resource as `[Resource: <name>\nURI: <uri>\nDescription: ...\nMIME type: ...]`.

**Key Logic:** Strips the `root_dir` prefix from `file://` URIs so paths displayed to the model are relative to the workspace.

---

### `convert_embedded_resource_block_to_content_blocks(block: EmbeddedResourceContentBlock) -> list[dict[str, str]]`

**Purpose:** Convert an ACP embedded resource block (inline content) to a text block.

**Parameters:**
- `block`: ACP embedded resource block. Its `.resource` must have either a `.text` or `.blob` attribute.

**Return Value:**
- For text resources: `[{"type": "text", "text": "[Embedded <mime> resource: <text>"}]`
- For blob resources: `[{"type": "text", "text": "[Embedded resource: data:<mime>;base64,<blob>]"}]`

**Raises:** `ValueError` if neither `.text` nor `.blob` is present.

---

### `extract_command_types(command: str) -> list[str]`

**Purpose:** Parse a shell command string and extract a list of security-relevant "command signatures". Handles `&&`-separated commands, pipe-separated segments, and sensitive command families with their subcommands or flags.

**Parameters:**
- `command`: Full shell command string (may include `&&`, `|`, quoted arguments).

**Return Value:** A list of command signature strings. For sensitive commands, signatures include enough detail to enforce fine-grained allowlists without over-permissioning (e.g., `"python -m pytest"`, not just `"python"`).

**Key Logic:**
- Splits on `&&` first, then on `|` within each segment.
- Uses `shlex.split()` for reliable tokenization.
- Dispatches to per-command handlers for: `python`, `python3`, `node`, `npm`, `npx`, `yarn`, `pnpm`, `uv`.
- Non-sensitive commands return just the base command name.

**Signature extraction strategy per command family:**

| Command | Strategy |
|---------|----------|
| `python`/`python3` | `-m <module>` → `"python -m <module>"`; `-c` → `"python -c"`; script → `"python"` |
| `node` | `-e`/`-p` → `"node -e"` / `"node -p"`; script → `"node"` |
| `npm`/`yarn`/`pnpm` | `run <script>` → `"npm run <script>"`; others → `"npm <subcommand>"` |
| `uv` | `run <tool>` → `"uv run <tool>"`; others → `"uv <subcommand>"` |
| `npx` | `"npx <package>"` |
| others | base command name only |

**Examples:**
```python
extract_command_types("npm install")              # ["npm install"]
extract_command_types("cd /p && python -m pytest")# ["cd", "python -m pytest"]
extract_command_types("ls -la | grep foo")        # ["ls", "grep"]
```

---

### `truncate_execute_command_for_display(command: str) -> str`

**Purpose:** Truncate a command string to 120 characters for display, appending `"..."` if truncated.

**Parameters:**
- `command`: Raw command string.

**Return Value:** The (possibly truncated) command string.

---

### `format_execute_result(command: str, result: str) -> str`

**Purpose:** Format a shell command and its raw output into a structured markdown block for display.

**Parameters:**
- `command`: The shell command that was executed.
- `result`: Raw result string from the execute tool, which may contain exit-code annotations like `[Command succeeded with exit code 0]` and `[Output was truncated...]`.

**Return Value:** A markdown-formatted string with `**Command:**`, `**Output:**`, `**Status:**`, and optionally a truncation note.

**Key Logic:** Parses the `result` line-by-line to separate actual output from metadata lines, then reassembles into a formatted display.

## Important Imports and Dependencies

| Import | Source | Purpose |
|--------|--------|---------|
| `shlex` | stdlib | Safe shell command tokenization |
| `acp.schema` (TYPE_CHECKING) | `agent-client-protocol` | Type annotations for ACP block types |
