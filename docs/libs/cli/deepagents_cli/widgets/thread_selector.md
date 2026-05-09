# `libs/cli/deepagents_cli/widgets/thread_selector.py`

> Interactive thread selector screen for /threads command.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from rich.cells import cell_len`

- `from textual.binding import Binding, BindingType`

- `from textual.color import Color as TColor`

- `from textual.containers import Horizontal, Vertical, VerticalScroll`

- `from textual.content import Content`

- `from textual.css.query import NoMatches`

- `from textual.fuzzy import Matcher`

- `from textual.message import Message`

- `from textual.screen import ModalScreen`

- `from textual.style import Style as TStyle`

- `from textual.widgets import Checkbox, Input, Static`

- `from deepagents_cli import theme`


## Functions and classes

### `_get_format_fns()`

Return cached `(format_path, format_relative_timestamp, format_timestamp)`.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_apply_column_width(cell: Static, key: str, column_widths: Mapping[str, int | None])`

Apply an explicit width to a table cell when one is configured.

Additional notes from the source docstring:

```text
Args:
    cell: The cell widget to size.
    key: Column key for the cell.
    column_widths: Effective column widths for the current table state.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_active_sort_key(sort_by_updated: bool)`

Return the active timestamp field used for sorting.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_visible_column_keys(columns: dict[str, bool])`

Return visible columns in the on-screen order.

Additional notes from the source docstring:

```text
Args:
    columns: Column visibility settings keyed by column name.

Returns:
    Visible column keys in display order.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_collapse_whitespace(value: str)`

Normalize a text value onto a single display line.

Additional notes from the source docstring:

```text
Args:
    value: Raw text to display in a single cell.

Returns:
    The input text collapsed to a single line.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_truncate_value(value: str, width: int | None)`

Trim text to fit a fixed-width column.

Additional notes from the source docstring:

```text
Args:
    value: Raw cell text.
    width: Maximum column width, or `None` for no truncation.

Returns:
    The possibly truncated display string.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_format_column_value(thread: ThreadInfo, key: str, *, relative_time: bool=False)`

Return the display text for one thread column.

Additional notes from the source docstring:

```text
Args:
    thread: Thread metadata for the row.
    key: Column key to format.
    relative_time: Use relative timestamps instead of absolute.

Returns:
    Formatted display text for the column cell.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_format_header_label(key: str)`

Return the rendered header label for a column.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_header_cell_classes(key: str, *, sort_key: str)`

Return CSS classes for a header cell.

Additional notes from the source docstring:

```text
Args:
    key: Column key for the header cell.
    sort_key: Currently active sort column.

Returns:
    Space-delimited classes for the header cell widget.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `ThreadOption`

A clickable thread option in the selector.

Methods worth reading inside this class:

- `compose(self)`: Compose the row cells.

- `_cursor_text(self)`: Return the cursor indicator for the row.

- `set_selected(self, selected: bool)`: Update row selection styling without rebuilding the row.

- `on_click(self, event: Click)`: Handle click on this option.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `DeleteThreadConfirmScreen`

Confirmation modal shown before deleting a thread.

Methods worth reading inside this class:

- `compose(self)`: Compose the confirmation dialog.

- `action_confirm(self)`: Confirm deletion.

- `action_cancel(self)`: Cancel deletion.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `ThreadSelectorScreen`

Modal dialog for browsing and resuming threads.

Additional notes from the source docstring:

```text
Displays recent threads with keyboard navigation, fuzzy search,
configurable columns, and delete support.

Returns a `thread_id` string on selection, or `None` on cancel.
```

Methods worth reading inside this class:

- `_switch_id(column_key: str)`: Return the DOM id for a column toggle switch.

- `_switch_column_key(switch_id: str | None)`: Extract the column key from a switch id.

- `_sync_selected_index(self)`: Select the current thread when it exists in the loaded rows.

- `_build_title(self, thread_url: str | None=None)`: Build the title, optionally with a clickable thread ID link.

- `_build_help_text(self)`: Build the footer help text for the selector.

- `_effective_thread_limit(self)`: Return the resolved thread limit for display purposes.

- `_format_sort_toggle_label(self)`: Return the control-panel sort label for the toggle switch.

- `_get_filter_input(self)`: Return the cached search input widget.

- `_filter_focus_order(self)`: Return the cached tab order for filter controls in the side panel.

- `compose(self)`: Compose the screen layout.

- `on_mount(self)`: Fetch threads, configure border for ASCII terminals, and build the list.

- `_start_thread_load(self)`: Launch the thread-load worker after the initial layout pass.

- `on_input_changed(self, event: Input.Changed)`: Filter threads as user types.

- `on_input_submitted(self, event: Input.Submitted)`: Handle Enter key when filter input is focused.

- `on_key(self, event: Key)`: Return focus to search when letters are typed from other controls.

- `_collapse_search_selection(self)`: Place the search cursor at the end without an active selection.

- `on_checkbox_changed(self, event: Checkbox.Changed)`: Route sort, relative-time, and column-visibility checkbox changes.

- `_update_filtered_list(self)`: Update filtered threads based on search text using fuzzy matching.

- `_compute_column_widths(self)`: Return effective widths for the current table state.

- `_get_search_text(thread: ThreadInfo)`: Build searchable text from thread fields.

- `_schedule_filter_and_rebuild(self)`: Queue a filter + rebuild, coalescing rapid keystrokes.

- `_filter_and_build(self)`: Run fuzzy filtering in a thread then rebuild the list.

- `_compute_filtered(query: str, threads: list[ThreadInfo], sort_by_updated: bool)`: Compute filtered thread list off the main thread.

- `_schedule_list_rebuild(self)`: Queue a list rebuild, coalescing rapid updates.

- `_pending_checkpoint_fields(self)`: Return which visible checkpoint-derived fields still need loading.

- `_populate_visible_checkpoint_details(self)`: Load any still-missing checkpoint-derived fields for visible columns.

- `_schedule_checkpoint_enrichment(self)`: Schedule one checkpoint-enrichment pass for missing row fields.

- `_threads_match(old: list[ThreadInfo], new: list[ThreadInfo])`: Check whether two thread lists have the same IDs and checkpoints in order.

- `_load_threads(self)`: Load thread rows first, then kick off background enrichment.

- `_load_checkpoint_details(self)`: Populate checkpoint-derived thread fields in one background pass.

- `_refresh_cell_labels(self)`: Update visible cell text in-place without rebuilding the DOM.

- `_resolve_thread_url(self)`: Start exclusive background worker to resolve LangSmith thread URL.

- `_fetch_thread_url(self)`: Resolve the LangSmith URL and update the title with a clickable link.

- `_show_mount_error(self, detail: str)`: Display an error message inside the thread list and refocus.

- `_build_list(self, *, recompute_widths: bool=True)`: Build the thread option widgets.

- `_create_option_widgets(self)`: Build option widgets from filtered threads without mounting.

- `_scroll_selected_into_view(self)`: Scroll selected option into view without animation.

- `_update_help_widgets(self)`: Update visible header and help text after state changes.

- `_schedule_header_rebuild(self)`: Queue a header rebuild to reflect column/sort changes.

- `_rebuild_header(self)`: Replace header cells to match current visible columns.

- `_apply_sort(self)`: Sort filtered threads by the active sort key.

- `_move_selection(self, delta: int)`: Move selection by delta, updating only the affected rows.

- `action_move_up(self)`: Move selection up.

- `action_move_down(self)`: Move selection down.

- `_visible_page_size(self)`: Return the number of thread options that fit in one visual page.

- `action_page_up(self)`: Move selection up by one visible page.

- `action_page_down(self)`: Move selection down by one visible page.

- `action_select(self)`: Confirm the highlighted thread and dismiss the selector.

- `action_focus_next_filter(self)`: Move focus through the filter and column-toggle controls.

- `action_focus_previous_filter(self)`: Move focus backward through the filter and column-toggle controls.

- `action_toggle_sort(self)`: Toggle sort between updated_at and created_at.

- `_persist_sort_order(self, order: str)`: Save sort-order preference to config, notifying on failure.

- `action_delete_thread(self)`: Show delete confirmation for the highlighted thread.

- `is_delete_confirmation_open(self)`: Return whether the delete confirmation overlay is visible.

- `_on_delete_confirmed(self, thread_id: str, confirmed: bool | None)`: Handle the result from the delete confirmation modal.

- `_handle_delete_confirm(self, thread_id: str)`: Execute thread deletion after confirmation.

- `on_click(self, event: Click)`: Open Rich-style hyperlinks on single click.

- `on_thread_option_clicked(self, event: ThreadOption.Clicked)`: Handle click on a thread option.

- `action_cancel(self)`: Cancel the selection.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
