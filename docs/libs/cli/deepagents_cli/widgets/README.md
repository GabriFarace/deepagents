# `libs/cli/deepagents_cli/widgets/`

> Textual leaf widgets and modal screens not owned by the core widget pass.

## Position in the system

`app.py` composes these widgets around the streaming runtime documented
elsewhere. The files here mostly own local presentation state: selectors,
history, notification views, onboarding/launch prompts, auth credential screens,
loading indicators, and auxiliary popups.

## Files Covered Here

- [`agent_selector.md`](./agent_selector.md) documents the modal for choosing an
  agent profile.
- [`ask_user.md`](./ask_user.md) documents the Textual UI for LangGraph
  `ask_user` interrupts.
- [`auth.md`](./auth.md) documents credential add/delete/list screens.
- [`autocomplete.md`](./autocomplete.md) documents slash-command and `@file`
  completion.
- [`history.md`](./history.md) documents prompt history navigation state.
- [`launch_init.md`](./launch_init.md) documents first-run launch setup screens.
- [`loading.md`](./loading.md) documents spinner/loading widgets.
- [`mcp_viewer.md`](./mcp_viewer.md) documents MCP tool browsing.
- [`model_selector.md`](./model_selector.md) documents model search, auth
  status, and selection UI.
- [`notification_center.md`](./notification_center.md),
  [`notification_detail.md`](./notification_detail.md), and
  [`notification_settings.md`](./notification_settings.md) document notification
  management screens.
- [`status.md`](./status.md) documents the status bar and model label.
- [`theme_selector.md`](./theme_selector.md) documents theme switching.
- [`thread_selector.md`](./thread_selector.md) documents thread list browsing,
  sorting, deletion, and selection.
- [`update_available.md`](./update_available.md) documents the update prompt.
- [`welcome.md`](./welcome.md) documents the welcome banner and footers.

## Gotchas

Widget contracts include Python message classes, Textual IDs/classes, and TCSS
selectors. A rename that looks local can still break bindings in `app.py` or
styles in [`../app.tcss.md`](../app.tcss.md).
