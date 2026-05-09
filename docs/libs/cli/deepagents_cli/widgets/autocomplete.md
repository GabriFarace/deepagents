# `libs/cli/deepagents_cli/widgets/autocomplete.py`

> Trigger-based completion for slash commands and `@` file mentions.

## Position in the system

The chat input widget delegates completion behavior to this module. The module
does not own Textual rendering directly; instead it talks through a
`CompletionView` protocol so the input widget can render, clear, and replace
completion ranges while the controllers own matching and keyboard state.

## Imports and module-level state

The file uses `subprocess` only for `git ls-files`, `shutil.which()` to locate
`git`, `SequenceMatcher` for fuzzy scoring, and
`deepagents_cli.project_utils.find_project_root()` to scope file suggestions.
`MAX_SUGGESTIONS` caps popup size, while `_MIN_SLASH_FUZZY_SCORE` and
`_MIN_DESC_SEARCH_LEN` prevent noisy command matches.

## Functions and classes

### `_get_git_executable()`

Returns the absolute `git` executable path from `shutil.which()`, or `None`.
`_get_project_files()` uses this so it can prefer Git's tracked-file view when
available and fall back to filesystem walking otherwise.

### `CompletionResult`

String enum describing how a controller handled a key event: ignored, handled,
or submit. The manager returns this to the caller so the chat input knows
whether to let Textual continue processing the key or to suppress the default
behavior.

### `CompletionView`

Protocol implemented by the input view. It provides three operations:
render suggestions, clear suggestions, and replace a text range with the chosen
completion. Keeping this as a protocol lets both slash and file completion share
the same rendering bridge.

### `CompletionController`

Protocol implemented by individual completion modes. Each controller decides
whether it can handle the current text/cursor, reacts to text changes, consumes
navigation/selection keys, and resets its own state.

### `SlashCommandController`

Owns `/` command completion. It stores the command list, current suggestions,
and selected index; `update_commands()` lets runtime-discovered skill commands
be merged into the static command registry.

`can_handle()` accepts input that begins with `/`. `on_text_changed()` builds a
search string up to the cursor, hides suggestions once a space indicates the
command has been chosen, and otherwise ranks command names, descriptions, and
hidden keywords. `_score_command()` prioritizes prefix and substring matches
before falling back to fuzzy matching.

`on_key()` handles arrows, tab, enter, and escape. `_move_selection()` wraps the
highlight through the suggestion list, and `_apply_selected_completion()` replaces
the command prefix with the selected command plus a trailing space.

### `_get_project_files(root)`

Returns candidate files for `@` completion. It prefers `git ls-files` so ignored
and untracked noise does not dominate, then falls back to walking the project
tree if Git is unavailable or fails.

### `_fuzzy_score(query, candidate)`

Scores file candidates for fuzzy matching. Exact, basename, substring, and
ordered-character matches receive stronger scores than weak similarity matches,
which keeps nearby likely paths at the top of the popup.

### `_is_dotpath(path)` and `_path_depth(path)`

Small ranking helpers for file suggestions. Dotpaths and deeply nested paths are
penalized so common project files remain discoverable before hidden or noisy
entries.

### `_fuzzy_search(query, candidates, max_results=MAX_SUGGESTIONS)`

Applies `_fuzzy_score()` across file candidates, sorts by score and path quality,
and returns `(label, description)` rows for the view. This is the shared search
primitive behind `FuzzyFileController`.

### `FuzzyFileController`

Owns `@file` completion. It keeps a cached file list, lazily refreshes it from
the detected project root, and exposes `warm_cache()` so the UI can prime file
suggestions without blocking the first completion interaction.

`can_handle()` checks whether the cursor is inside an `@` mention. On text
changes it extracts the active mention query, searches cached files, and renders
suggestions. Its key handling mirrors slash completion: navigation changes the
selected row, selection replaces the active mention range, and escape clears the
popup.

### `MultiCompletionManager`

Coordinates multiple completion controllers. `on_text_changed()` chooses the
first controller that can handle the current input and resets the previously
active controller when control switches. `on_key()` forwards keys only to the
active controller, and `reset()` clears all completion state.

## Gotchas

Completion replacement is cursor-range sensitive. Any change to query extraction
must be tested with cursor positions in the middle of text, not only at the end
of the input.
