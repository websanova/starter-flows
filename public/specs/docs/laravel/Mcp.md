# MCP

Status: done
Updated: 2026-10-14

The API supports two logins. Sanctum signs in the App's Users, and Passport signs in MCP Clients through OAuth.

Laravel's docs present the two packages as a choice between one or the other, but the API needs both, and they were not designed to share a guard, a token trait or a user model. The Implementation section walks through how they are wired together and why each piece is set up the way it is.

## Implementation

### 1. Front end URLs

The env variables `FRONTEND_APP_URL` and `FRONTEND_ADMIN_URL` hold the base URL of each front end. App and Admin are separate front ends, likely on separate domains with separate auth sessions, so a User signed in to one is not signed in to the other. The API needs the address of each, and the two values drive the authorize page addresses in the OAuth metadata.

### 2. Passport tables and keys

Passport's migrations create its tables, and a key pair signs the access tokens. Locally the keys come from `passport:keys`. In production the deploy script creates them, as described in the [Forge doc](/docs/laravel/Forge).

### 3. Guard and user model

The `api` guard uses the Passport driver on an `oauth_users` provider. The model behind that provider, OAuthUser, reads the same users table as User.

Two models are needed because Sanctum and Passport each ship a token trait with the same method names, and each guard hands its own token type to the same method on the user. One class cannot carry both traits, so User keeps Sanctum and OAuthUser carries Passport.

### 4. Config

The `mcp.servers` list names each API MCP Server and its path, and `passport.authorization_endpoints` names each front end's authorize page under the same names, built from the step 1 URLs. The metadata points MCP Clients there instead of at the API, which serves no html.

The lifetimes of the Passport tokens issued to MCP Clients are set at boot in the `AppServiceProvider`. An access token lasts one hour because the [MCP authorization spec](https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization) says authorization servers should issue short-lived access tokens, so a leaked token does limited damage. Passport's default is one year, so the override is what meets that guidance. The short lifetime also means an MCP Client refreshes during normal use, as in step 16.1 of the [MCP Login flow](/flows/auth/McpLogin).

The spec also requires refresh tokens to rotate for public clients, and MCP Clients are public clients. Passport's refresh grant already revokes the old refresh token and issues a new one, so no setup is needed. The spec gives no refresh token lifetime, so the 30 days is a choice with no spec basis.

One scope, `mcp:use`, comes from the MCP package and goes to every registered client. Nothing checks the scope yet, so a token issued through one API MCP Server is accepted by the other as long as the User passes that server's checks.

### 5. Routes

The MCP package loads `routes/ai.php` itself. MCP Clients are given only the API MCP Server URL, so the file begins with `Mcp::oauthRoutes()`, which adds the package's discovery and registration routes. Three routes registered after that replace the package's versions of the same address.

The two metadata routes are overridden because the package assumes one issuer per MCP service with it's own custom set of endpoints for each. They are moved into their own folder just for code clarity and separation.

Registration keeps the package's address and controller. Registration is public and creates a client on every call, so the route adds a throttle. The route also gets a name, because the authorization server metadata uses the name to build the registration endpoint.

### 6. Bridge middleware

After Passport authenticates the OAuthUser, a middleware loads the real User by id and sets the User on the guard. The verified, admin and tool checks then work unchanged, and a soft deleted account fails here with a 401.

### 7. MCP controllers

Two controllers answer the metadata routes from the step 4 config. The protected resource controller names the API MCP Server URL as its own issuer. The authorization server controller returns that server's front end authorize page, plus the token and registration endpoints.

Each API MCP Server is its own issuer because metadata names one authorize page and App and Admin are separate front ends. A new API MCP Server is one entry in each config list plus its route.

### 8. Consent

The file `routes/oauth.php` is loaded through the API routing setup, so the routes get the same middleware as `routes/api.php`.

The `GET /oauth/consent` route is public and validates the request through Passport's authorization server. A bad client or redirect_uri fails before the User is asked to log in, which is the order Passport's own authorize route uses.

The `POST /oauth/consent` route needs a signed in User who is verified and has updated their password. The route completes the request and returns the redirect address in the body. A User blocked from the API cannot hand an MCP Client a token that fails on every call.

Passport's own authorize routes keep the request in a web session, render a view and answer with redirects. The App sends every parameter again with each call and needs JSON, so these two endpoints are used instead. One pair serves App and Admin because nothing in consent depends on role.

### 9. Login and token

The App signs the User in with the normal Sanctum login. The code exchange and the refresh go to Passport's token endpoint, which already answers JSON and needs no replacement.

### 10. Using it

The connected MCP Client calls `/mcp` or `/admin/mcp` with the access token. The request passes the guard, the bridge middleware and the usual middleware. A User without the Admin role gets a 403 on the Admin API MCP Server.
