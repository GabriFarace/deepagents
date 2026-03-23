# Workflow: `deepagents-example.yml` — Deep Agents Example

## Overview

An example/reference workflow that demonstrates how to integrate the Deep Agents action into a repository. It allows authorized team members to invoke the AI agent by mentioning `@deepagents` in PR comments, and the agent will read context, make code changes, and respond — all within the GitHub Actions environment.

## Trigger

- `issue_comment` events (type: `created`) — triggered when a comment is posted on an issue or PR
- `pull_request_review_comment` events (type: `created`) — triggered on inline PR review comments
- `workflow_dispatch` with a `prompt` input — for manual invocations

Concurrent runs for the same PR/issue are cancelled (only the latest `@deepagents` mention survives).

## Security Gate

The agent only runs when triggered by a comment if **all** of:
1. The comment body contains `@deepagents`
2. The commenter has `OWNER`, `MEMBER`, or `COLLABORATOR` association
3. The event is either a PR review comment or a comment on a PR (not a bare issue)

This prevents unauthorized users from invoking the agent.

## Jobs

### `deepagents`

Runs on `ubuntu-latest`.

**Permissions:** `contents: write`, `issues: write`, `pull-requests: write`

| Step | Description |
|---|---|
| Resolve PR number | Extracts PR number from event payload |
| Acknowledge trigger | Adds a rocket reaction to the triggering comment (best-effort) |
| Get PR head SHA | Fetches the PR's head commit SHA and branch name via `gh pr view` |
| Checkout | Checks out the PR branch so the agent can commit to it directly |
| Build PR context prompt | Constructs a rich prompt including PR metadata, diff, comments, reviews, and the trigger comment |
| Run Deep Agents | Invokes `langchain-ai/deepagents@main` with the constructed prompt |

## Prompt Construction

The "Build PR context prompt" step assembles structured XML context for the agent:

- `<pull-request>`: title, author, state, branches, body
- `<changed-files>`: list of modified files
- `<pull-request-comments>`: up to 20 issue comments
- `<pull-request-reviews>`: up to 10 review summaries
- `<review-comments>`: up to 30 inline review comments
- `<trigger-comment>`: the specific comment that triggered this workflow run

The agent is instructed to resolve the trigger comment with minimal changes and post a summary comment when done.

## Secrets Used

| Secret | Purpose |
|---|---|
| `ANTHROPIC_API_KEY` | API key for Claude models |

The example uses `claude-sonnet-4-6` with skills from `langchain-ai/langchain-skills`. Alternative providers (OpenAI, Google) are shown in comments.

## Notes

- This is a **reference/example** workflow — it demonstrates the pattern for using Deep Agents in CI.
- The `skills_repo` input installs additional skills into the agent's `.deepagents/skills/` directory.
- For production use, adapt this workflow by adjusting the security gate, model choice, and prompt as needed.
