# Forge

Status: done
Updated: 2026-10-14

## MCP / Passport

The API MCP Servers sign in through Passport, which needs a pair of signing keys on the server. Add the key line to the deploy script after the migrate step.

```bash
[ -f storage/oauth-private.key ] || $FORGE_PHP artisan passport:keys
```

The check generates the keys on the first deploy and skips them on every deploy after. Without the check, a deploy that finds existing keys fails. The keys live in `storage/`, which has to persist between deploys.
