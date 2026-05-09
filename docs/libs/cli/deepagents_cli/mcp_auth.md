# `libs/cli/deepagents_cli/mcp_auth.py`

> OAuth login flow and token storage for MCP servers.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Imports and module-level state

Important imports in this file connect it to these adjacent systems:

- `from mcp.client.auth import OAuthClientProvider, TokenStorage`

- `from mcp.shared.auth import OAuthClientInformationFull, OAuthToken`


## Functions and classes

### `_DeviceCodeResponse`

RFC 8628 §3.2 device-authorization response payload.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `McpServerSpec`

Parsed MCP server config entry.

Additional notes from the source docstring:

```text
All keys are optional at the type level because `mcpServers` entries
are validated shape-first by `_validate_server_config` rather than by
the type system. This TypedDict documents the accepted shape for
readers and static checkers — validate the fields at use sites before
relying on them.
```

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `resolve_headers(headers: dict[str, str], *, server_name: str | None=None)`

Resolve `${VAR}` env-var references in header values.

Additional notes from the source docstring:

```text
Args:
    headers: Raw header mapping from MCP config.
    server_name: Optional server name for error messages.

Returns:
    A new dict with env-var references resolved to current values.

Raises:
    TypeError: If a header value is not a string.
    RuntimeError: If a `${VAR}` reference points to an unset env var.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_interpolate(s: str, *, header: str, server_name: str | None)`

Expand `${VAR}` references in `s` against the current environment.

Additional notes from the source docstring:

```text
Args:
    s: Raw header value.
    header: Header name, used in error messages.
    server_name: Owning server name for error messages.

Returns:
    Interpolated string.

Raises:
    RuntimeError: If a referenced env var is unset.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_tokens_dir()`

Return `~/.deepagents/.state/mcp-tokens/`.

Additional notes from the source docstring:

```text
The deferred import lets tests redirect token storage into a temp
directory by patching `deepagents_cli.model_config.DEFAULT_STATE_DIR`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_token_file_stem(server_name: str, server_url: str | None)`

Return a path-safe storage stem for this server identity.

Additional notes from the source docstring:

```text
Safety of the stem depends on `server_name` already having passed
`_SERVER_NAME_RE` in `_validate_server_config` — the URL is hashed
to a hex digest, so only the server name can carry path separators.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `FileTokenStorage`

File-backed `TokenStorage` under `~/.deepagents/.state/mcp-tokens/`.

Methods worth reading inside this class:

- `path(self)`: Return the on-disk token file path for this server.

- `get_tokens(self)`: Return the stored `OAuthToken`, or `None` if none is persisted.

- `set_tokens(self, tokens: OAuthToken)`: Persist `tokens` to disk, preserving any stored client info.

- `get_client_info(self)`: Return the stored client registration, or `None` if none is persisted.

- `set_client_info(self, client_info: OAuthClientInformationFull)`: Persist `client_info` to disk, preserving any stored tokens.

- `set_tokens_and_client_info(self, tokens: OAuthToken, client_info: OAuthClientInformationFull)`: Persist tokens and client info in a single atomic write.

- `_read(self)`: This loader reads configuration, metadata, or persisted state and applies the module's fallback behavior for missing or malformed input.

- `_write(self, data: dict)`: This writer updates disk or runtime state after a user action or completed flow.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `MCPReauthRequiredError`

Raised when an MCP server needs interactive re-authentication.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `_make_reauth_required_handlers(server_name: str)`

Return OAuth handlers that refuse to prompt and raise instead.

Additional notes from the source docstring:

```text
Used in non-interactive server mode so that a missing or expired token
surfaces as `MCPReauthRequiredError` rather than hanging on `input()`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_make_paste_back_handlers(*, extra_auth_params: dict[str, str] | None=None)`

Create paste-back redirect and callback handlers for OAuth.

Additional notes from the source docstring:

```text
Args:
    extra_auth_params: Extra query params to append to the auth URL.

Returns:
    A tuple of `(redirect_handler, callback_handler)`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_append_query_params(url: str, params: dict[str, str])`

Return `url` with `params` replacing any same-named query keys.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `build_oauth_provider(*, server_name: str, server_url: str, storage: TokenStorage, extra_auth_params: dict[str, str] | None=None, interactive: bool=True)`

Construct an `OAuthClientProvider` for an MCP server.

Additional notes from the source docstring:

```text
Args:
    server_name: MCP server name used in re-auth messages.
    server_url: Remote MCP server URL.
    storage: Token storage implementation for this server.
    extra_auth_params: Optional query params for the interactive auth URL.
    interactive: Whether the provider may prompt on stdin.

Returns:
    A configured `OAuthClientProvider`.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_run_device_flow(*, device_code_url: str, token_url: str, client_id: str, scope: str | None=None)`

Run OAuth 2.0 Device Authorization Grant and return the token.

Additional notes from the source docstring:

```text
Args:
    device_code_url: Provider endpoint that issues a device + user code.
    token_url: Provider endpoint to poll for the access token.
    client_id: Registered OAuth client ID.
    scope: Optional space-delimited scope string.

Returns:
    The issued OAuth access token payload.

Raises:
    RuntimeError: If the device flow fails, times out, or the provider
        returns an unexpected HTTP status on the device-code request.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `find_reauth_required(exc: BaseException)`

Find an `MCPReauthRequiredError` anywhere inside `exc`'s tree.

Additional notes from the source docstring:

```text
Walks `exceptions` (for `ExceptionGroup`), then `__cause__` and
`__context__`, tracking visited nodes to terminate on cyclic chains.

Args:
    exc: Root exception to inspect.

Returns:
    The nested `MCPReauthRequiredError`, or `None` if not present.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `_drive_handshake(connections: dict)`

Open a one-shot MCP session for `connections` to trigger OAuth handshake.

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

### `login(*, server_name: str, server_config: McpServerSpec)`

Drive OAuth login for `server_name`, persisting tokens on success.

Additional notes from the source docstring:

```text
Args:
    server_name: Name of the configured MCP server.
    server_config: Parsed server config for that entry.

Raises:
    ValueError: If `server_config` isn't an OAuth http/sse server.
    RuntimeError: If header env-var interpolation fails, the device
        flow fails or times out, or the OAuth handshake aborts.
```

In the larger CLI flow, this block is intentionally scoped: callers pass already-scoped inputs, and the function either returns a normalized value or performs the module-specific side effect described above. Keep its return shape stable because the importing core files usually do not re-validate it.

## Gotchas

MCP helpers frequently handle credentials, token files, or trust state. Preserve URL matching, fingerprinting, and storage boundaries when editing them.
