# Starter Flows

Feature flow specs for the starter projects.

| Flow | Status | Updated |
| ---- | ------ | ------- |
| [Subscription Create - Stripe (Payment Element, intent on init)](flows/subscriptions/stripe/create.md) | draft | 2026-08-16 |
| [Subscription Create - Stripe (Hosted Checkout)](flows/subscriptions/stripe/reference/create-hosted.md) | reference | 2026-08-16 |
| [Subscription Create - Stripe (Embedded Checkout)](flows/subscriptions/stripe/reference/create-embedded.md) | reference | 2026-08-16 |
| [Subscription Create - Stripe (Payment Element, deferred intent)](flows/subscriptions/stripe/reference/create-deferred.md) | reference | 2026-08-16 |
| [Billing Update - Stripe (Payment Element)](flows/billing/stripe/update.md) | draft | 2026-08-17 |

A `reference/` directory holds strategies that were evaluated and not taken. The chosen one is the flow file beside it.

## Projects

API

* [Starter Laravel API](https://github.com/websanova/starter-laravel-api)

APP

* [Starter Vue SPA](https://github.com/websanova/starter-vue-spa)

## How It Works

Each feature gets one flow file in `flows/`: logic, states, rules, edge cases, error handling, decisions, mermaid diagrams. Flows are implementation-agnostic - a Stripe subscription flow is the same in Laravel, Rails, Django, Vue, React, or mobile. Only the library choices differ, and those are a code session problem.

Work the flow out here first, then paste it into a session in the target code repo:

* `Execute this flow.`
* `Review this flow for missing parts to add to our code.`
* `Review this flow against the code and report what needs updating.`

Status: `draft` (being worked out), `approved` (ready to execute), `implemented` (executed in at least one repo), `reference` (research, not executable). Section template in [CLAUDE.md](CLAUDE.md).

## Docker

Markdown viewer for `flows/` at http://localhost:8088.

```bash
docker compose up -d      # start
docker compose down       # stop
docker compose restart    # reload after editing viewer/nginx.conf
docker compose ps         # status
docker compose logs -f    # tail nginx logs
```

Everything is bind mounted read-only. Edits to `flows/*.md` and `viewer/index.html` are live, no restart. Only `nginx.conf` needs one.
