# Starter Flows

Feature flow specs and setup docs for the starter projects.

| Doc | Status | Updated |
| --- | ------ | ------- |
| [Terms](docs/conventions/Terms.md) | done | 2026-08-31 |
| [Status](docs/conventions/Status.md) | done | 2026-09-04 |
| [Docker Setup](docs/setup/DockerSetup.md) | done | 2026-09-02 |

| Flow | Status | Updated |
| ---- | ------ | ------- |
| [Subscription Create - Stripe (Checkout Sessions, Payment Element)](flows/subscription/stripe/Create.md) | done | 2026-09-04 |
| [Subscription Cancel - Stripe](flows/subscription/stripe/Cancel.md) | done | 2026-09-04 |
| [Subscription Resume - Stripe](flows/subscription/stripe/Resume.md) | done | 2026-09-06 |
| [Subscription Update - Stripe](flows/subscription/stripe/Update.md) | done | 2026-09-07 |
| [Subscription Guards](flows/subscription/Guards.md) | wip | 2026-09-03 |
| [Payment Method Update - Stripe (Payment Element)](flows/payment-method/stripe/Update.md) | done | 2026-09-06 |
| [Payment Method Delete - Stripe](flows/payment-method/stripe/Delete.md) | done | 2026-09-03 |
| [Address Update - Stripe (Address Element)](flows/address/stripe/Update.md) | done | 2026-09-03 |
| [Account Delete](flows/account/Delete.md) | wip | 2026-09-07 |

| Ref | Status | Updated |
| --- | ------ | ------- |
| [Subscription Create - Stripe (Hosted Checkout)](refs/subscription/stripe/CreateHosted.md) | ref | 2026-09-01 |
| [Subscription Create - Stripe (Embedded Checkout)](refs/subscription/stripe/CreateEmbedded.md) | ref | 2026-09-01 |

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

Markdown viewer for `flows/`, `docs/` and `refs/` at http://localhost:8088.

```bash
docker compose up -d      # start
docker compose down       # stop
docker compose restart    # reload after editing viewer/nginx.conf
docker compose ps         # status
docker compose logs -f    # tail nginx logs
```

Everything is bind mounted read-only. Edits to `flows/*.md`, `docs/*.md`, `refs/*.md` and `viewer/index.html` are live, no restart. Only `nginx.conf` needs one.

## Deploy

If you want to deploy the flows publicly, just serve the viewer directory and create a couple aliases for the `flows`, `docs` and `refs` folders.

```nginx
server {
    add_header Cache-Control "no-store" always;

    location /flows/ {
        alias /path/to/app/dir/flows/;
        autoindex on;
        autoindex_format json;
        default_type text/markdown;
    }

    location /docs/ {
        alias /path/to/app/dir/docs/;
        autoindex on;
        autoindex_format json;
        default_type text/markdown;
    }

    location /refs/ {
        alias /path/to/app/dir/refs/;
        autoindex on;
        autoindex_format json;
        default_type text/markdown;
    }
}
```
