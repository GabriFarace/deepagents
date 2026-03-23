# `widgets/loading.py`

## High-Level Purpose

This module defines the `LoadingWidget` — an animated loading indicator displayed while the agent is thinking or executing. It shows a spinner animation, status text, elapsed time, and an interrupt hint.

## Classes

### `Spinner`

A stateful animated spinner using charset-appropriate frames.

**Methods:**

- `__init__()` — Initialize spinner at position 0.
- `next_frame() -> str` — Advance the spinner to the next frame and return the character.
- `current_frame() -> str` — Return the current frame without advancing.

**Key Logic:** Frames are loaded lazily from `get_glyphs().spinner_frames` to support both Unicode (⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏) and ASCII (`|/-\`) modes. The position wraps around using modulo.

### `LoadingWidget`

**Inherits from:** `textual.widgets.Static`

An animated loading indicator displayed while the agent is processing.

**Display format:** `<spinner> Thinking... (3s, esc to interrupt)`

**Default CSS:**
```css
LoadingWidget {
    height: auto;
    padding: 0 1;
    margin-top: 1;
}
```

**Inner widgets:**
- `.loading-container` — Horizontal container
- `.loading-spinner` — Animated spinner character (primary color)
- `.loading-status` — Status text like "Thinking..." or "Offloading..." (primary color)
- `.loading-hint` — Elapsed time and interrupt hint (muted color)

**Key Methods:**

- `on_mount() -> None` — Starts a timer to update the spinner every 100ms.
- `update_status(status: SpinnerStatus) -> None` — Updates the status text (e.g., from `"Thinking"` to `"Offloading"`).
- `stop() -> None` — Stops the animation timer and hides the widget.
- `_tick() -> None` — Called every 100ms to advance the spinner and update elapsed time.

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.widgets.Static` | textual | Base widget |
| `textual.containers.Horizontal` | textual | Container for spinner layout |
| `get_glyphs` | `deepagents_cli.config` | Glyph selection (unicode vs ASCII) |
| `SpinnerStatus` | `deepagents_cli._session_stats` | Status type alias |
