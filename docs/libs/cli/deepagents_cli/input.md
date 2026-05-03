# `libs/cli/deepagents_cli/input.py`

## High-Level Purpose

`input.py` handles pre-processing of user input before it is sent to the agent. Its two responsibilities are: parsing `@file` mentions embedded in the user's message (and attaching file contents), and tracking pasted or dragged media (images, PDFs) so they can be included in the message as binary content.

---

## Key Functions

### `parse_file_mentions(text, backend) → tuple[str, list[FileAttachment]]`

Scans the user's message for `@path/to/file` patterns. For each mention:
1. Resolves the path relative to the project root
2. Reads the file content via the backend
3. Returns the modified message text (mentions may be replaced or annotated) plus a list of `FileAttachment` objects

`FileAttachment` contains:
- `path` — the original path
- `content` — file content as string or base64
- `encoding` — `"utf-8"` or `"base64"`

The attachments are appended to the human message as separate content blocks before sending to the LangGraph server.

### `MediaTracker`

Tracks media files (images, PDFs, audio) that the user has pasted or dragged into the TUI.

**Key methods:**

- `add(path_or_bytes, mime_type)` — registers a media item
- `pop_all() → list[MediaItem]` — returns and clears all pending media items
- Called from `ChatInput` on paste events; flushed in `app.py` when the message is submitted

---

## Architecture Notes

**Why pre-process in the TUI?** Attaching file contents in the TUI layer (rather than having the agent call `read_file`) avoids a round-trip through the server for large files. The user explicitly referenced them — attaching them upfront is efficient.

**@mention syntax:** The `@` convention is inspired by Claude Code's file mention system. The regex matches absolute and relative paths, and paths quoted with backticks. Paths that don't resolve or aren't readable are left as-is in the message text (the agent will see the literal `@path` string).

---

## See Also

- [app.md](app.md) — calls `parse_file_mentions()` before enqueuing a message
- [widgets/chat_input.md](widgets/chat_input.md) — `ChatInput` uses `MediaTracker` for paste events
