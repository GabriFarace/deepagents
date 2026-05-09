# `libs/cli/deepagents_cli/widgets/model_selector.py`

> Textual modal for browsing, filtering, selecting, and setting default chat
> models.

## Position in the system

`app.py` opens this modal when the user asks to change models. The screen reads
model/provider metadata from the CLI model configuration layer, renders grouped
provider sections, annotates each row with auth readiness, and returns either a
`(model_spec, provider)` tuple or `None` when cancelled.

## Imports and module-level state

The widget depends on Textual screen, input, container, and message primitives;
theme helpers for styled `Content`; and model-config helpers for provider
metadata and credential state. Its bindings map keyboard navigation, selection,
default-setting, paging, tab completion, and cancellation to action methods.

## Functions and classes

### `ModelOption`

Row widget for one selectable model. The constructor records the displayed
`provider:model` spec, provider key, selected/current/default flags, auth status,
and optional model status. The `provider` property exposes the provider for
selection results.

`Clicked` is the row's message type. `on_click()` stops the click event and
posts `Clicked`, letting the parent screen centralize selection behavior instead
of mutating modal state inside each row.

### `ModelSelectorScreen`

Modal screen that owns model search and selection state. `__init__()` receives
the current model, configured default model, and any model metadata needed to
render the grouped list. `_find_current_model_index()` computes the initial
highlighted row.

`compose()` builds the modal layout: search input, scrollable model list, and
footer/help area. `_load_model_data()` gathers provider and model records, while
`_curate_models()` reduces that data into the list shown to users. `on_mount()`
focuses the input, renders the initial list, and scrolls the current/default
selection into view.

Search flows through `on_input_changed()`, `on_input_submitted()`, and
`_update_filtered_list()`. `_update_display()` rebuilds grouped provider headers
and option rows from the filtered list. `_format_auth_indicator()`,
`_format_option_label()`, `_format_footer()`, and `_get_model_status()` keep
rendering rules out of the selection logic.

Navigation and selection live in `_move_selection()`, `action_move_up()`,
`action_move_down()`, `action_page_up()`, `action_page_down()`,
`action_tab_complete()`, `action_select()`, and `_visible_page_size()`.
`_select_with_auth_check()` prevents choosing models whose provider credentials
block startup, while `action_set_default()` persists a new default model through
the configuration layer.

`_restore_help_text()`, `action_cancel()`, and `_dismiss_with_result()` finish
the modal lifecycle. The result is passed back to the caller only through
`dismiss()`, which keeps caller-side handling consistent for click, keyboard, and
cancel paths.

## Gotchas

The file uses modern Python syntax, so older parsers may fail even though the
runtime package supports it. UI changes should be checked with missing,
configured, implicit, managed, and unknown credential states because row styling
and selection blocking depend on those distinctions.
