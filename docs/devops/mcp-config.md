# MCP Configuration

## Overview

`.mcp.json` configures [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) servers for use with the deepagents agent and compatible AI assistants. MCP servers expose additional tools and context sources that agents can call at runtime.

## Location

`/.mcp.json`

## Configured Servers

### `docs-langchain`

| Field | Value |
|---|---|
| Type | `http` |
| URL | `https://docs.langchain.com/mcp` |

Provides access to LangChain documentation. Agents can query this server to retrieve up-to-date documentation, API references, and guides from the LangChain documentation site.

### `reference-langchain`

| Field | Value |
|---|---|
| Type | `http` |
| URL | `https://reference.langchain.com/mcp` |

Provides access to the LangChain API reference. Agents can query this server to look up class definitions, method signatures, and module structures.

## Usage

This configuration is automatically picked up by the deepagents CLI when running in the repository root. The MCP servers are available as additional tool sources for agents during task execution.

## Notes

- Both servers use the `http` transport type, connecting to remote MCP endpoints over HTTPS.
- No authentication is required for these public documentation servers.
- The MCP protocol allows agents to discover available tools from each server at runtime.
