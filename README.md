# Starter Flows

Feature flow specs and setup docs for the starter projects.

| Doc | Status | Updated |
| --- | ------ | ------- |
| [Terms](docs/Terms.md) | current | 2026-08-31 |
| [Docker Setup](docs/DockerSetup.md) | WIP | 2026-08-31 |

| Flow | Status | Updated |
| ---- | ------ | ------- |
| [Subscription Create - Stripe (Checkout Sessions, Payment Element)](flows/subscriptions/stripe/Create.md) | draft | 2026-08-31 |
| [Subscription Create - Stripe (Hosted Checkout)](flows/subscriptions/stripe/reference/CreateHosted.md) | reference | 2026-08-31 |
| [Subscription Create - Stripe (Embedded Checkout)](flows/subscriptions/stripe/reference/CreateEmbedded.md) | reference | 2026-08-31 |
| [Subscription Cancel - Stripe](flows/subscriptions/stripe/Cancel.md) | draft | 2026-08-17 |
| [Subscription Resume - Stripe](flows/subscriptions/stripe/Resume.md) | draft | 2026-08-17 |
| [Billing Payment Method Update - Stripe (Payment Element)](flows/billing/stripe/PaymentMethodUpdate.md) | draft | 2026-08-17 |
| [Billing Payment Method Delete - Stripe](flows/billing/stripe/PaymentMethodDelete.md) | draft | 2026-08-17 |
| [Billing Address Update - Stripe](flows/billing/stripe/AddressUpdate.md) | draft | 2026-08-17 |
| [Subscription Guards](flows/subscriptions/Guards.md) | WIP | 2026-08-30 |

A `reference/` directory holds strategies that were evaluated and not taken. The chosen one is the flow file beside it.

## Projects

API

* [Starter Laravel API](https://github.com/websanova/starter-laravel-api)

APP

* [Starter Vue SPA](https://github.com/websanova/starter-vue-spa)

## How It Works

Each feature gets one flow file in `flows/`, holding a description, the shared terms, the requirements, the numbered steps, a mermaid diagram, notes and outstanding items. Anything that is not a feature gets a doc in `docs/`, which has no fixed structure. Both are implementation-agnostic, so a Stripe subscription flow is the same in Laravel, Rails, Django, Vue, React, or mobile. Only the library choices differ, and those are a code session problem.

Every term used in either is defined once in [docs/Terms.md](docs/Terms.md) and copied word for word wherever it appears.

Work the flow out here first, then paste it into a session in the target code repo:

* `Execute this flow.`
* `Review this flow for missing parts to add to our code.`
* `Review this flow against the code and report what needs updating.`

Flow status: `WIP` (ideas being gathered), `draft` (being worked out), `approved` (ready to execute), `implemented` (executed in at least one repo), `reference` (research, not executable).

Doc status: `WIP` (being written), `current` (describes the setup as it is). Section template in [CLAUDE.md](CLAUDE.md).

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
