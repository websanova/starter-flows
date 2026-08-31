# Starter Flows

Feature flow specs and setup docs for the starter projects.

| Doc | Status | Updated |
| --- | ------ | ------- |
| [Terms](docs/conventions/Terms.md) | done | 2026-08-31 |
| [Status](docs/conventions/Status.md) | done | 2026-08-31 |
| [Docker Setup](docs/setup/DockerSetup.md) | wip | 2026-08-31 |

| Flow | Status | Updated |
| ---- | ------ | ------- |
| [Subscription Create - Stripe (Checkout Sessions, Payment Element)](flows/subscriptions/stripe/Create.md) | done | 2026-08-31 |
| [Subscription Create - Stripe (Hosted Checkout)](flows/subscriptions/stripe/CreateHosted.md) | ref | 2026-08-31 |
| [Subscription Create - Stripe (Embedded Checkout)](flows/subscriptions/stripe/CreateEmbedded.md) | ref | 2026-08-31 |
| [Subscription Cancel - Stripe](flows/subscriptions/stripe/Cancel.md) | draft | 2026-08-17 |
| [Subscription Resume - Stripe](flows/subscriptions/stripe/Resume.md) | draft | 2026-08-17 |
| [Billing Payment Method Update - Stripe (Payment Element)](flows/billing/stripe/PaymentMethodUpdate.md) | draft | 2026-08-17 |
| [Billing Payment Method Delete - Stripe](flows/billing/stripe/PaymentMethodDelete.md) | draft | 2026-08-17 |
| [Billing Address Update - Stripe](flows/billing/stripe/AddressUpdate.md) | draft | 2026-08-17 |
| [Subscription Guards](flows/subscriptions/Guards.md) | wip | 2026-08-30 |

A `ref` flow is a strategy that was evaluated and not taken. The one that was taken sits beside it.

## Projects

API

* [Starter Laravel API](https://github.com/websanova/starter-laravel-api)

APP

* [Starter Vue SPA](https://github.com/websanova/starter-vue-spa)

## How It Works

Each feature gets one flow file in `flows/`, holding a description, the shared terms, the requirements, the numbered steps, a mermaid diagram, notes and outstanding items. Anything that is not a feature gets a doc in `docs/`, which has no fixed structure. Both are implementation-agnostic, so a Stripe subscription flow is the same in Laravel, Rails, Django, Vue, React, or mobile. Only the library choices differ, and those are a code session problem.

Every term used in either is defined once in [docs/conventions/Terms.md](docs/conventions/Terms.md) and copied word for word wherever it appears.

Work the flow out here first, then paste it into a session in the target code repo:

* `Execute this flow.`
* `Review this flow for missing parts to add to our code.`
* `Review this flow against the code and report what needs updating.`

Every status keyword is defined in [docs/conventions/Status.md](docs/conventions/Status.md). Section template in [CLAUDE.md](CLAUDE.md).

## Docker

Markdown viewer for `flows/` and `docs/` at http://localhost:8088.

```bash
docker compose up -d      # start
docker compose down       # stop
docker compose restart    # reload after editing viewer/nginx.conf
docker compose ps         # status
docker compose logs -f    # tail nginx logs
```

Everything is bind mounted read-only. Edits to `flows/*.md`, `docs/*.md` and `viewer/index.html` are live, no restart. Only `nginx.conf` needs one.
