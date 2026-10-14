# MCP

Status: done
Updated: 2026-10-14

The API supports two logins. Sanctum signs in the App's Users, and Passport signs in MCP Clients through OAuth.

Laravel's docs present the two packages as a choice between one or the other, but the API needs both, and they were not designed to share a guard, a token trait or a user model. The Implementation section walks through how they are wired together and why each piece is set up the way it is.

## Implementation

### 1. Front end URLs

The env variables `FRONTEND_APP_URL` and `FRONTEND_ADMIN_URL` hold the base URL of each front end. App and Admin are separate front ends, likely on separate domains with separate auth sessions, so a User signed in to one is not signed in to the other. The API needs the address of each, and the two values drive the authorize page addresses in the OAuth metadata.

### 2. Passport tables and keys

Passport's migrations create its tables. Locally the signing keys come from `passport:keys`. In production the deploy script creates them, as described in the [Forge doc](/docs/laravel/Forge).

### 3. Guard and user model

The `api` guard uses the Passport driver on an `oauth_users` provider. The model behind that provider, OAuthUser, reads the same users table as User.

Two models are needed because Sanctum and Passport each ship a token trait with the same method names, and each guard hands its own token type to the same method on the user. One class cannot carry both traits, so User keeps Sanctum and OAuthUser carries Passport.

### 4. Config

The `mcp.servers` list names the API MCP Servers by name and path, where `app` is `mcp` and `admin` is `admin/mcp`. The `passport.authorization_endpoints` list names each front end's authorize page under the same names. The `mcp.redirect_domains` list limits where a registered MCP Client may redirect, which is claude.ai, claude.com and localhost.

The lifetimes of the Passport tokens issued to MCP Clients are set at boot in the `AppServiceProvider`. An access token lasts one hour and a refresh token lasts 30 days. The short access token means an MCP Client refreshes during normal use, as in step 16.1 of the [MCP Login flow](/flows/auth/McpLogin). Passport's default access token lasts one year, so no refresh would ever run.

One scope, `mcp:use`, comes from the MCP package and goes to every registered client. Nothing checks the scope yet.

### 5. Routes

The MCP package loads `routes/ai.php` itself. The file begins with `Mcp::oauthRoutes()`, which adds the package's default discovery and registration.

Three routes then replace the package's versions of the same address, which are protected resource metadata, authorization server metadata, and registration with a throttle added. The package assumes one issuer and an authorize page on the API itself, and its registration is public and creates a client on every call. The protected resource route keeps the package's route name, because the package builds the 401 header's metadata address from the name.

The two API MCP Servers follow, guarded by `auth:api`, the bridge middleware from step 6, and the same stack as the matching API routes.

### 6. Bridge middleware

After Passport authenticates, a middleware loads the real User by id and sets the User on the guard. The verified, admin and tool checks then work unchanged, and a soft deleted account fails here with a 401.

### 7. MCP controllers

Two controllers answer the metadata routes from the step 4 config. The protected resource controller names the API MCP Server URL as its own issuer. The authorization server controller returns that server's front end authorize page, plus the token and registration endpoints.

Each API MCP Server is its own issuer because metadata names one authorize page and App and Admin are separate front ends. A new API MCP Server is one entry in each config list plus its route.

### 8. Consent

The file `routes/oauth.php` is loaded through the API routing setup, so the routes get the same middleware as `routes/api.php`.

The `GET /oauth/consent` route is public and validates the request through Passport's authorization server. A bad client or redirect_uri fails before the User is asked to log in, which is the order Passport's own authorize route uses.

The `POST /oauth/consent` route needs a signed in User who is verified and has updated their password. The route completes the request and returns the redirect address in the body. A User blocked from the API cannot hand an MCP Client a token that fails on every call.

Passport's own authorize routes keep the request in a web session, render a view and answer with redirects. The App sends every parameter again with each call and needs JSON, so these two endpoints replace them. One pair serves App and Admin because nothing in consent depends on role.

### 9. Login and token

The App signs the User in with the normal Sanctum login. The code exchange and the refresh go to Passport's stock token endpoint.

### 10. Using it

The connected MCP Client calls `/mcp` or `/admin/mcp` with the access token. The request passes the guard, the bridge middleware and the usual middleware. A User without the Admin role gets a 403 on the Admin API MCP Server.
