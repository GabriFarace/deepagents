# `libs/cli/deepagents_cli/notifications.py`

> Registry of pending actionable notifications.

## Position in the system

This helper is imported by the CLI core rather than being an entry point itself. It keeps one peripheral concern isolated so `main.py`, `app.py`, and the server/client lifecycle files can stay focused on orchestration.

## Functions and classes

### `ActionId`

Stable identifiers for notification actions dispatched by the app.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `NotificationAction`

One button/action row in the notification modal.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `MissingDepPayload`

Typed payload for a missing-dependency notification.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `UpdateAvailablePayload`

Typed payload for an update-available notification.

The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `PendingNotification`

A single notice waiting for user action.

Additional notes from the source docstring:

```text
Immutable value object: the registry owns the
key-to-toast-identity binding (see `NotificationRegistry`) so
external callers cannot corrupt click-routing indices by mutating
notifications after construction.
```

Methods worth reading inside this class:

- `__post_init__(self)`: Enforce basic invariants at construction time.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

### `NotificationRegistry`

In-memory store of pending notifications.

Additional notes from the source docstring:

```text
Instance-scoped (one per app) so test apps don't pollute each other.
Owns the bidirectional key-to-toast-identity binding so callers
cannot accidentally desynchronize the click-routing indices.
```

Methods worth reading inside this class:

- `add(self, notification: PendingNotification)`: Register a new notification or replace an existing one with the same key.

- `remove(self, key: str)`: Remove a notification by key.

- `get(self, key: str)`: Return the notification for *key*, or `None` when not registered.

- `bind_toast(self, key: str, toast_identity: str)`: Attach a Textual toast identity to an existing notification.

- `toast_identity_for(self, key: str)`: Return the toast identity bound to *key*, or `None`.

- `unbind_toast(self, toast_identity: str)`: Drop the binding for *toast_identity*, if any.

- `key_for_toast(self, toast_identity: str)`: Return the registered key for *toast_identity*, or `None`.

- `is_actionable_toast(self, toast_identity: str)`: Return whether a click on *toast_identity* should open the modal.

- `list_all(self)`: Return all pending notifications in insertion order.

- `clear(self)`: Remove all entries. Primarily useful for tests.


The class owns local state for this module and limits mutation to the storage, UI, provider, or parsing boundary named in the class documentation.

## Gotchas

Most helpers in this area are called from core CLI files that are documented separately. Check both sides of the call before changing return shapes or exception behavior.
