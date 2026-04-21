# Claude Code Boilerplate

My starting point for new projects with [Claude Code](https://claude.com/claude-code). Ships with a few MCP servers and skills I reach for often.

## Setup

```sh
cp .mcp.sample.json .mcp.json
```

Fill in the tokens in `.mcp.json`, then open the project in Claude Code. Drop any server you don't need.

## What's inside

**MCP servers** — `playwright`, `atlassian` (Jira + Confluence), `github`, `gitlab`.

**Skills** — `atlassian`, `browser`, `pr-mr-review`, `thomas-guidelines`.

## Requires

`npx` for most servers, `uvx` for the Atlassian one.

## Note

`.mcp.json` holds personal tokens and is gitignored. Don't commit it.
