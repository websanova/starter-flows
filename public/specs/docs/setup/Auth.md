# Auth

Status: wip
Updated: 2026-10-07

The API runs two auth setups. Sanctum covers the App with straightforward bearer authentication. Passport is Laravel's OAuth implementation, which MCP clients such as claude.ai and Claude Desktop use to connect to the MCP servers.

## Diagrams

Sanctum.

```mermaid
sequenceDiagram
    participant A as App
    participant P as API
    A->>P: Log in with email and password
    P-->>A: Sanctum token
    A->>P: Call route with Bearer token
```

Passport.

```mermaid
sequenceDiagram
    participant C as MCP client
    participant U as User
    participant A as App
    participant P as API
    C->>P: Call MCP server with no token
    P-->>C: 401 pointing to the authorization server
    C->>P: Fetch discovery metadata and register as a client
    C->>U: Open browser at the API authorize URL
    U->>P: Request authorize
    P-->>U: Redirect to App login and consent page
    U->>A: Log in and approve
    A->>P: Send approval
    P-->>U: Redirect to MCP client with authorization code
    U->>C: Deliver authorization code
    C->>P: Exchange code for access token
    P-->>C: Passport access token
    C->>P: Call MCP server with Bearer token
```
