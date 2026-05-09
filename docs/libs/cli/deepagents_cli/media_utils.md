# `libs/cli/deepagents_cli/media_utils.py`

> Utilities for handling image and video media from clipboard and files.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `_get_executable(name: str)`

Get full path to an executable using shutil.which().

Additional notes from the source docstring:

```text
Args:
    name: Name of the executable to find

Returns:
    Full path to executable, or None if not found.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `ImageData`

Represents a pasted image with its base64 encoding.

Methods worth reading inside this class:

- `to_message_content(self)`: Convert to LangChain message content format.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `VideoData`

Represents a pasted video with its base64 encoding.

Methods worth reading inside this class:

- `to_message_content(self)`: Convert to LangChain `VideoContentBlock` format.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `get_clipboard_image()`

Attempt to read an image from the system clipboard.

Additional notes from the source docstring:

```text
Supports macOS via `pngpaste` or `osascript`.

Returns:
    ImageData if an image is found, None otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_image_from_path(path: pathlib.Path)`

Read and encode an image file from disk.

Additional notes from the source docstring:

```text
Args:
    path: Path to the image file.

Returns:
    `ImageData` when the file is a valid image, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_detect_video_format(data: bytes)`

Detect video MIME subtype from magic bytes.

Additional notes from the source docstring:

```text
Args:
    data: Raw file bytes (at least 12 bytes for reliable detection).

Returns:
    MIME subtype (e.g. "mp4", "webm") or `None` if unrecognized.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_video_from_path(path: pathlib.Path)`

Read and encode a video file from disk.

Additional notes from the source docstring:

```text
Args:
    path: Path to the video file.

Returns:
    `VideoData` when the file is a valid video, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `get_media_from_path(path: pathlib.Path)`

Try to load a file as an image first, then as a video.

Additional notes from the source docstring:

```text
Args:
    path: Path to the media file.

Returns:
    `ImageData` or `VideoData` if the file is valid media, otherwise `None`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_get_macos_clipboard_image()`

Get clipboard image on macOS using pngpaste or osascript.

Additional notes from the source docstring:

```text
First tries pngpaste (faster if installed), then falls back to osascript.

Returns:
    ImageData if an image is found, None otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_get_clipboard_via_osascript()`

Get clipboard image via osascript using a temp file.

Additional notes from the source docstring:

```text
osascript outputs data in a special format that can't be captured as raw binary,
so we write to a temp file instead.

Returns:
    ImageData if an image is found, None otherwise.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `encode_to_base64(data: bytes)`

Encode raw bytes to a base64 string.

Additional notes from the source docstring:

```text
Args:
    data: Raw bytes to encode.

Returns:
    Base64-encoded string.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `create_multimodal_content(text: str, images: list[ImageData], videos: list[VideoData] | None=None)`

Create multimodal message content with text, images, and videos.

Additional notes from the source docstring:

```text
Args:
    text: Text content of the message
    images: List of ImageData objects
    videos: Optional list of VideoData objects

Returns:
    List of content blocks in LangChain message format.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
