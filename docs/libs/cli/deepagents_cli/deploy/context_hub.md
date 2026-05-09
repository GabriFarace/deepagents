# `libs/cli/deepagents_cli/deploy/context_hub.py`

> ContextHubBackend: Store files in a LangSmith Hub agent repo (persistent).

## Position in the system

This file belongs to the `deepagents deploy` path. The deploy package reads a project layout, validates `deepagents.toml`, bundles prompts, memories, skills, MCP config, optional frontend assets, and emits files that `langgraph deploy` can run.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from deepagents.backends.protocol import BackendProtocol, EditResult, FileData, FileDownloadResponse, FileInfo, FileUploadResponse, GlobResult, GrepMatch, GrepResult, LsResult, ReadResult, WriteResult`

- `from deepagents.backends.utils import create_file_data, perform_string_replacement, slice_read_response`


## Functions and classes

### `ContextHubBackend`

Backend that stores files in a LangSmith Hub agent repo (persistent).

Methods worth reading inside this class:

- `_load_tree(self)`: Fetch the file tree; missing repos are treated as empty (first commit creates them).

- `_ensure_cache(self)`: Load the file tree if not yet loaded.

- `get_linked_entries(self)`: Return linked-entry paths mapped to their repo handles.

- `has_prior_commits(self)`: Return True if the hub repo already exists with at least one commit.

- `_commit(self, files: dict[str, str])`: Push ``files`` as one commit; update the cache on success.

- `_strip_prefix(path: str)`: This helper performs one narrow step for the surrounding module while keeping normalization, error handling, or side-effect policy in one place.

- `read(self, file_path: str, offset: int=0, limit: int=2000)`: Read file content for the requested line range.

- `write(self, file_path: str, content: str)`: Commit ``content`` to ``file_path``.

- `edit(self, file_path: str, old_string: str, new_string: str, replace_all: bool=False)`: Replace ``old_string`` with ``new_string``; fails on multiple matches unless ``replace_all=True``.

- `ls(self, path: str='/')`: List immediate files and subdirectories under ``path`` (non-recursive).

- `grep(self, pattern: str, path: str | None=None, glob: str | None=None)`: Search contents for ``pattern`` (optional ``path`` / ``glob`` filters).

- `glob(self, pattern: str, path: str='/')`: Return files matching ``pattern`` (``path`` unused — flat namespace).

- `upload_files(self, files: list[tuple[str, bytes]])`: Upload text files in one commit; non-UTF-8 inputs rejected per file.

- `download_files(self, paths: list[str])`: Download files as raw bytes. Missing paths return ``file_not_found``.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Deploy helpers usually write files, read environment variables, or shell out through command helpers. Keep validation errors explicit because deployment failures otherwise surface late inside LangGraph Cloud tooling.
