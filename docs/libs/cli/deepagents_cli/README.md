# `libs/cli/deepagents_cli/`

> CLI package documentation index for the peripheral files covered in this pass.

## Scope

This directory mirrors the in-scope helper, widget, deploy, skills, MCP provider,
integration, prompt, and support files under `libs/cli/deepagents_cli/`.

The core CLI orchestration files are intentionally excluded here because they are
owned by the parent documentation pass: `main.py`, `app.py`, server lifecycle
files, `remote_client.py`, command registry, sessions, hooks, core MCP tools,
agent/config/model files, non-interactive mode, and the key message/input/tool
widgets.

## Reading Order

1. Start with helper surfaces that many other files call:
   [`project_utils.md`](./project_utils.md), [`_git.md`](./_git.md),
   [`_env_vars.md`](./_env_vars.md), [`_server_config.md`](./_server_config.md),
   [`auth_store.md`](./auth_store.md), and [`file_ops.md`](./file_ops.md).
2. Read user-facing CLI helpers:
   [`ui.md`](./ui.md), [`output.md`](./output.md), [`input.md`](./input.md),
   [`clipboard.md`](./clipboard.md), [`editor.md`](./editor.md), and
   [`formatting.md`](./formatting.md).
3. Read extension surfaces:
   [`skills/README.md`](./skills/README.md),
   [`mcp_providers/README.md`](./mcp_providers/README.md),
   [`integrations/README.md`](./integrations/README.md), and
   [`deploy/README.md`](./deploy/README.md).
4. Read the Textual leaf screens under [`widgets/README.md`](./widgets/README.md).
5. Finish with behavior-shaping text files:
   [`system_prompt.md`](./system_prompt.md),
   [`default_agent_prompt.md`](./default_agent_prompt.md), and
   [`built_in_skills/README.md`](./built_in_skills/README.md).

## Notes

Generated frontend assets under `deploy/frontend_dist/assets/` are not documented
file-by-file. They are build artifacts; the deploy docs cover how they are copied
and configured.
