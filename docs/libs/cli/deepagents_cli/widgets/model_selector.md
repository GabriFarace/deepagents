# `widgets/model_selector.py`

## High-Level Purpose

This module defines the `ModelSelectorScreen` — an interactive modal dialog for the `/model` slash command. It lists all available models grouped by provider, supports fuzzy search filtering, shows credential status, and allows the user to select a model or set it as the default.

## Classes

### `ModelOption`

**Inherits from:** `textual.widgets.Static`

A clickable model option in the selector list.

**Constructor parameters:**
| Parameter | Type | Description |
|---|---|---|
| `label` | `str \| Content` | Display content |
| `model_spec` | `str` | Model spec in `provider:model` format |
| `provider` | `str` | Provider name |
| `index` | `int` | Index in the filtered list |
| `has_creds` | `bool \| None` | Credential status: `True` confirmed, `False` missing, `None` unknown |

**Inner Message: `Clicked`**

Posted when the user clicks a model option.

| Attribute | Type | Description |
|---|---|---|
| `model_spec` | `str` | The clicked model specification |
| `provider` | `str` | The provider name |
| `index` | `int` | Index of the clicked option |

### `ModelSelectorScreen`

**Inherits from:** `textual.screen.ModalScreen[str | None]`

A modal screen for model selection.

**Bindings:**
| Key | Action | Description |
|---|---|---|
| `escape` | `cancel` | Close without selecting |
| `up / k` | `move_up` | Navigate up |
| `down / j` | `move_down` | Navigate down |
| `enter` | `select` | Select highlighted model |
| `d` | `set_default` | Set selected model as default |
| `ctrl+d` | `clear_default` | Clear the persisted default model |

**Constructor:**

```python
ModelSelectorScreen(
    current_model: str | None = None,
    default_model: str | None = None
)
```

**Parameters:**
- `current_model`: Currently active model spec for highlighting.
- `default_model`: Persisted default model spec.

**Behavior:**
1. Loads all available models via `get_available_models()`.
2. Loads model profiles via `get_model_profiles()`.
3. Checks credentials for each provider via `has_provider_credentials()`.
4. Displays models with provider grouping, fuzzy search via Textual's `Matcher`.
5. Shows a `[D]` badge next to the default model.
6. Shows a `[no key]` warning for providers without credentials.
7. Returns the selected model spec string or `None` if cancelled.

**Typing in the search `Input` filters the model list in real time.**

## Important Imports and Dependencies

| Import | Source | Purpose |
|---|---|---|
| `textual.screen.ModalScreen` | textual | Modal dialog base |
| `textual.fuzzy.Matcher` | textual | Fuzzy text matching |
| `textual.widgets.Input` | textual | Search input |
| `ModelConfig`, `get_available_models`, `get_model_profiles` | `model_config` | Model data loading |
| `has_provider_credentials`, `save_default_model`, `clear_default_model` | `model_config` | Credential checks and persistence |
