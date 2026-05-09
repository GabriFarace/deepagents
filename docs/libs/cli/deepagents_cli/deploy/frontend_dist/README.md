# `libs/cli/deepagents_cli/deploy/frontend_dist/`

> Pre-built frontend assets copied into deploy bundles when `[frontend].enabled`
> is true.

## Position in the system

`deploy/bundler.py` copies this directory into the build output and rewrites the
`window.__DEEPAGENTS_CONFIG__` placeholder in `index.html` with runtime
configuration derived from `deepagents.toml` and environment variables.

## Files

- `index.html` contains the runtime config placeholder and loads the bundled
  assets.
- `logo-light.svg` and `logo-dark.svg` are static logo assets.
- `assets/` contains minified build outputs. They are intentionally not
  documented one by one in this pass.

## Gotchas

If the placeholder in `index.html` changes, `_copy_frontend_dist()` will fail
with an explicit out-of-sync error and ask for the frontend bundle to be rebuilt.
