# `libs/cli/deepagents_cli/input.py`

> Input handling utilities including image/video tracking and file mention parsing.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from rich.markup import escape as escape_markup`

- `from deepagents_cli.config import console`

- `from deepagents_cli.media_utils import ImageData, VideoData`


## Functions and classes

### `ParsedPastedPathPayload`

Unified parse result for dropped-path payload detection.

Additional notes from the source docstring:

```text
Attributes:
    paths: Resolved file paths parsed from the input payload.
    token_end: End index (exclusive) of the parsed leading token when the
        payload starts with a path followed by trailing text.

        `None` means the entire payload was parsed as path-only content.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `MediaTracker`

Track pasted images and videos in the current conversation.

Methods worth reading inside this class:

- `add_media(self, data: ImageData | VideoData, kind: MediaKind)`: Add a media item and return its placeholder text.

- `add_image(self, image_data: ImageData)`: Add an image and return its placeholder text.

- `add_video(self, video_data: VideoData)`: Add a video and return its placeholder text.

- `get_media(self, kind: MediaKind)`: Get all tracked media of a given type.

- `get_images(self)`: Get all tracked images.

- `get_videos(self)`: Get all tracked videos.

- `clear(self)`: Clear all tracked media and reset counters.

- `sync_to_text(self, text: str)`: Retain only media still referenced by placeholders in current text.

- `_sync_kind_images(self, text: str)`: Sync image list to surviving placeholders in text.

- `_sync_kind_videos(self, text: str)`: Sync video list to surviving placeholders in text.

- `_max_placeholder_id(items: list[ImageData] | list[VideoData], pattern: re.Pattern[str], fallback_count: int)`: Compute next ID from the highest surviving placeholder.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `parse_file_mentions(text: str)`

Extract `@file` mentions and return the text with resolved file paths.

Additional notes from the source docstring:

```text
Parses `@file` mentions from the input text and resolves them to absolute
file paths. Files that do not exist or cannot be resolved are excluded with
a warning printed to the console.

Email addresses (e.g., `user@example.com`) are automatically excluded by
detecting email-like characters before the `@` symbol.

Backslash-escaped spaces in paths (e.g., `@my\ folder/file.txt`) are
unescaped before resolution. Tilde paths (e.g., `@~/file.txt`) are expanded
via `Path.expanduser()`. Only regular files are returned; directories are
excluded.

This function does not raise exceptions; invalid paths are handled
internally with a console warning.

Args:
    text: Input text potentially containing `@file` mentions.

Returns:
    Tuple of (original text unchanged, list of resolved file paths that exist).
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `parse_pasted_file_paths(text: str)`

Parse a paste payload that may contain dragged-and-dropped file paths.

Additional notes from the source docstring:

```text
The parser is strict on purpose: it only returns paths when the entire paste
payload can be interpreted as one or more existing files. Any invalid token
falls back to normal text paste behavior by returning an empty list.

Supports common dropped-path formats:

- Absolute/relative paths
- POSIX shell quoting and escaping
- `file://` URLs

Args:
    text: Raw paste payload from the terminal.

Returns:
    List of resolved file paths, or an empty list when parsing fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `parse_pasted_path_payload(text: str, *, allow_leading_path: bool=False)`

Parse dropped-path payload variants through one entrypoint.

Additional notes from the source docstring:

```text
Parsing order is:
1. strict multi-path payload parsing (`parse_pasted_file_paths`)
2. single-path normalization/parsing (`parse_single_pasted_file_path`)
3. optional leading-path extraction (`extract_leading_pasted_file_path`)

Args:
    text: Input payload to parse.
    allow_leading_path: Whether to parse a leading path token followed by
        trailing prompt text.

Returns:
    Parsed payload details, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `parse_single_pasted_file_path(text: str)`

Parse and resolve a single pasted path payload.

Additional notes from the source docstring:

```text
Unlike `parse_pasted_file_paths`, this helper only accepts one path token
and is intended for fallback handling when a paste event carries a
single path representation.

Args:
    text: Raw pasted text payload.

Returns:
    Resolved path when payload is a single existing file, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `extract_leading_pasted_file_path(text: str)`

Extract and resolve a leading pasted path token from input text.

Additional notes from the source docstring:

```text
This is used for submit-time recovery when a user message starts with a
path token followed by additional prompt text.

Args:
    text: Input text to inspect.

Returns:
    Tuple of `(resolved_path, token_end_index)` or `None` when no valid
    leading file path token exists.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `normalize_pasted_path(text: str)`

Normalize pasted text that may represent a single filesystem path.

Additional notes from the source docstring:

```text
Supports:

- quoted and shell-escaped single paths
- `file://` URLs
- Windows drive-letter and UNC paths

Args:
    text: Raw pasted text payload.

Returns:
    Parsed `Path` if payload is a single path token, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_split_paste_line(line: str)`

Split a single pasted line into path-like tokens.

Additional notes from the source docstring:

```text
Args:
    line: A single line from the paste payload.

Returns:
    Parsed shell-like tokens, or an empty list when parsing fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_token_to_path(token: str)`

Convert a pasted token into a path candidate.

Additional notes from the source docstring:

```text
Args:
    token: A single shell-split token from the paste payload.

Returns:
    A parsed path candidate, or `None` when token parsing fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_leading_token_end(text: str)`

Return the end index of the first shell-like token.

Additional notes from the source docstring:

```text
Args:
    text: Input text beginning with a token.

Returns:
    End index (exclusive), or `None` when token parsing fails.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_extract_unquoted_leading_path_with_spaces(text: str)`

Extract a leading unquoted path that may contain spaces.

Additional notes from the source docstring:

```text
This fallback is intentionally POSIX-oriented (`/` and `~/`) because the
slash-command conflict it addresses is specific to inputs that begin with
`/`.

Args:
    text: Input text beginning with a potential path.

Returns:
    Tuple of `(resolved_path, token_end_index)` or `None` when no matching
    leading path prefix resolves to an existing file.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_normalize_windows_pasted_path(text: str)`

Return a `Path` for unquoted Windows drive/UNC path inputs.

Additional notes from the source docstring:

```text
Args:
    text: Potential Windows path input.

Returns:
    Parsed `Path` when `text` is Windows drive-letter or UNC style,
    otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_normalize_posix_pasted_path(text: str)`

Return a `Path` for likely POSIX absolute/home path payloads.

Additional notes from the source docstring:

```text
Some terminals paste dropped absolute paths with spaces as raw text without
quoting/escaping. In that case shell tokenization splits on spaces even
though the full payload is intended to be a single path.

Args:
    text: Potential POSIX path input.

Returns:
    Parsed `Path` when `text` looks like a raw POSIX absolute/home path,
    otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_existing_pasted_path(path: Path)`

Resolve a pasted path candidate to an existing file.

Additional notes from the source docstring:

```text
Performs an exact resolution first, then a Unicode-space-tolerant lookup.

Args:
    path: Parsed path candidate.

Returns:
    Resolved existing file path, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_normalize_unicode_spaces(text: str)`

Normalize Unicode lookalike spaces to ASCII spaces.

Additional notes from the source docstring:

```text
Args:
    text: Text to normalize.

Returns:
    Normalized text with Unicode-space variants converted to ASCII spaces.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_resolve_with_unicode_space_variants(path: Path)`

Resolve path by matching filename segments with Unicode space variants.

Additional notes from the source docstring:

```text
Args:
    path: Path candidate that may differ from disk by space code points.

Returns:
    Matching filesystem path, or `None` when no variant match exists.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
