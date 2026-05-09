# `libs/cli/deepagents_cli/widgets/auth.py`

> TUI screens for managing stored model-provider credentials.

## Position in the system

This widget sits on the Textual side of the CLI. `app.py` composes these screens and controls around streamed events from the LangGraph server, while the widget itself owns presentation, local interaction state, and the small event messages it emits upward.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from textual.binding import Binding, BindingType`

- `from textual.color import Color as TColor`

- `from textual.containers import Vertical`

- `from textual.content import Content`

- `from textual.screen import ModalScreen`

- `from textual.style import Style as TStyle`

- `from textual.widgets import Input, OptionList, Static`

- `from textual.widgets.option_list import Option`

- `from deepagents_cli import auth_store, theme`

- `from deepagents_cli.config import get_glyphs, is_ascii_mode`

- `from deepagents_cli.model_config import PROVIDER_API_KEY_ENV, PROVIDERS_DOCS_URL as _PROVIDERS_DOCS_URL, ModelConfig, ProviderAuthSource, clear_caches, get_available_models, get_credential_env_var, get_provider_auth_status, resolved_env_var_name`

- `from deepagents_cli.widgets._links import open_style_link`


## Functions and classes

### `AuthResult`

Outcome of an `AuthPromptScreen` interaction.

Additional notes from the source docstring:

```text
The three outcomes need to stay distinguishable because callers in the
recovery path retry the original failing operation only on `SAVED` —
retrying after `DELETED` would loop into the same missing-credentials
error indefinitely.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `DeleteCredentialConfirmScreen`

Confirmation overlay shown before clearing a stored credential.

Additional notes from the source docstring:

```text
Patterned on `DeleteThreadConfirmScreen` so the destructive prompt feels
consistent across the CLI. Always dismisses with `True` on confirm or
`False` on cancel; the caller does the actual delete.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the confirmation dialog.

- `action_confirm(self)`: Confirm deletion.

- `action_cancel(self)`: Cancel deletion.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AuthPromptScreen`

Modal that captures and persists an API key for one provider.

Additional notes from the source docstring:

```text
Dismissal values are members of `AuthResult` so callers in the recovery
path can distinguish "user just saved a key — retry the failed
operation" from "user just cleared their key — don't retry, that would
loop into the same error" from "user cancelled — leave state alone".
```

Methods worth reading inside this class:

- `compose(self)`: Compose the prompt.

- `on_mount(self)`: Apply ASCII border when needed.

- `on_input_submitted(self, event: Input.Submitted)`: Validate, persist, and dismiss.

- `action_cancel(self)`: Dismiss without saving.

- `action_delete_stored(self)`: Open the delete-confirmation overlay for the stored credential.

- `_on_delete_confirmed(self, confirmed: bool | None)`: Handle the result of the confirmation overlay.

- `_show_error(self, template: str, /, **substitutions: str)`: Render `template` via markup substitution in the inline error slot.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `AuthManagerScreen`

Modal that lists configured providers and lets the user manage keys.

Additional notes from the source docstring:

```text
Reachable via the `/auth` slash command. Always dismisses with `None`;
state changes are persisted by `AuthPromptScreen` and reflected by
re-rendering the option list when this screen is reopened or after a
save/delete completes.
```

Methods worth reading inside this class:

- `compose(self)`: Compose the manager.

- `_build_description(self)`: Build the description line with an inline docs hyperlink.

- `on_mount(self)`: Apply ASCII border when needed.

- `on_click(self, event: Click)`: Open style-embedded hyperlinks (the title `Docs` link).

- `on_option_list_option_selected(self, event: OptionList.OptionSelected)`: Open the prompt for the selected provider.

- `action_cancel(self)`: Close the manager.

- `action_cursor_down(self)`: Move the option-list cursor down.

- `action_cursor_up(self)`: Move the option-list cursor up.

- `_on_prompt_closed(self, _result: AuthResult | None)`: Refresh the option list once the prompt dismisses.

- `_refresh_options(self)`: Rebuild option labels from current store state.

- `_build_options_with_warning(self)`: Render the option list, returning a corruption warning if any.

- `_format_label(provider: str)`: Build a `Content` label for `provider` showing its credential source.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Widget classes are coupled to Textual message names, CSS classes, and screen lifecycle hooks. Changing identifiers here can break bindings in `app.py` or stylesheet selectors even when type checks pass.
