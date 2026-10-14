# MCP

Status: done
Updated: 2026-10-14

There are two API MCP Servers, the App server at `/mcp` and the Admin server at `/admin/mcp`, which is guarded by the Admin role. A User connects an MCP Client by signing in through the App and approving it. The steps behind the login are in the [MCP Login flow](/flows/auth/McpLogin).

## Claude Desktop

Open the Connectors settings, add a custom connector, give the connector a name and paste the API MCP Server URL.

| Name | URL |
| --- | --- |
| Starter App | `https://starter-laravel-api.websanova.com/mcp` |
| Starter Admin | `https://starter-laravel-api.websanova.com/admin/mcp` |

## Claude Code

Add the server from a terminal.

```bash
claude mcp add --transport http starter https://starter-laravel-api.websanova.com/mcp
claude mcp add --transport http starter-admin https://starter-laravel-api.websanova.com/admin/mcp
```
