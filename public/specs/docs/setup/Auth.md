# Auth

Status: wip
Updated: 2026-10-07

The API runs two auth setups. Sanctum covers the App with straightforward bearer authentication. Passport is Laravel's OAuth implementation, which MCP Clients such as claude.ai and Claude Desktop use to connect to the API MCP Servers. The step by step login is the [MCP Login flow](/flows/auth/McpLogin).

## Diagrams

Sanctum, for regular App usage with a bearer token.

```mermaid
sequenceDiagram
    participant A as App
    participant P as API
    A->>P: Log in with email and password
    P-->>A: Sanctum token
    A->>P: Call route with Bearer token
```

Passport, for MCP Clients with OAuth approval through the App.

```mermaid
sequenceDiagram
    participant C as MCP Client
    participant A as App
    participant P as API
    C->>A: Open browser at the App authorize page
    A->>P: Send the User's approval
    P-->>A: Authorization code
    A->>C: Redirect browser to the MCP Client with the code
```
