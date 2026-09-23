# Starter Flows

Feature flow specs and setup docs for the starter projects.

Part of the Starters, built at [Websanova](https://www.websanova.com).

| Doc | Status | Updated |
| --- | ------ | ------- |
| [Terms](viewer/specs/docs/conventions/Terms.md) | done | 2026-09-02 |
| [Status](viewer/specs/docs/conventions/Status.md) | done | 2026-09-04 |
| [Docker Setup](viewer/specs/docs/setup/DockerSetup.md) | done | 2026-09-02 |

| Flow | Status | Updated |
| ---- | ------ | ------- |
| [Subscription Create - Stripe (Checkout Sessions, Payment Element)](viewer/specs/flows/subscription/stripe/Create.md) | done | 2026-09-06 |
| [Subscription Cancel - Stripe](viewer/specs/flows/subscription/stripe/Cancel.md) | done | 2026-09-04 |
| [Subscription Resume - Stripe](viewer/specs/flows/subscription/stripe/Resume.md) | done | 2026-09-06 |
| [Subscription Update - Stripe](viewer/specs/flows/subscription/stripe/Update.md) | done | 2026-09-07 |
| [Subscription Guards](viewer/specs/flows/subscription/Guards.md) | wip | 2026-09-03 |
| [Payment Method Update - Stripe (Payment Element)](viewer/specs/flows/payment-method/stripe/Update.md) | done | 2026-09-06 |
| [Payment Method Delete - Stripe](viewer/specs/flows/payment-method/stripe/Delete.md) | done | 2026-09-03 |
| [Address Update - Stripe (Address Element)](viewer/specs/flows/address/stripe/Update.md) | done | 2026-09-03 |
| [Account Delete](viewer/specs/flows/account/Delete.md) | wip | 2026-09-07 |

| Ref | Status | Updated |
| --- | ------ | ------- |
| [Subscription Create - Stripe (Hosted Checkout)](viewer/specs/refs/subscription/stripe/CreateHosted.md) | ref | 2026-09-01 |
| [Subscription Create - Stripe (Embedded Checkout)](viewer/specs/refs/subscription/stripe/CreateEmbedded.md) | ref | 2026-09-01 |

## Projects

| Project | Repo | Demo |
| ------- | ---- | ---- |
| Starter Flows | [starter-flows](https://github.com/websanova/starter-flows) | [flows](https://starter-flows.websanova.com) |
| Starter Laravel API | [starter-laravel-api](https://github.com/websanova/starter-laravel-api) | [api](https://starter-laravel-api.websanova.com) |
| Starter Vue SPA | [starter-vue-spa](https://github.com/websanova/starter-vue-spa) | [app](https://starter-vue-spa-app.websanova.com), [admin](https://starter-vue-spa-admin.websanova.com) |

## How It Works

Each feature gets one flow file in `viewer/specs/flows/`, holding a description, the shared terms, the requirements, the numbered steps, a mermaid diagram, notes and outstanding items. Anything that is not a feature gets a doc in `viewer/specs/docs/`, which has no fixed structure. Both are implementation-agnostic, so a Stripe subscription flow is the same in Laravel, Rails, Django, Vue, React, or mobile. Only the library choices differ, and those are a code session problem.

Every term used in either is defined once in [viewer/specs/docs/conventions/Terms.md](viewer/specs/docs/conventions/Terms.md) and copied word for word wherever it appears.

Work the flow out here first, then paste it into a session in the target code repo:

* `Execute this flow.`
* `Review this flow for missing parts to add to our code.`
* `Review this flow against the code and report what needs updating.`

Every status keyword is defined in [viewer/specs/docs/conventions/Status.md](viewer/specs/docs/conventions/Status.md). Section template in [CLAUDE.md](CLAUDE.md).

The viewer sidebar reads [viewer/files.json](viewer/files.json), a hand kept list of every file with its status and updated date. A new flow, doc or ref needs a line there as well as a row in the tables above.

## Docker

Markdown viewer for `viewer/specs/` at http://localhost:8088.

```bash
docker compose up -d      # start
docker compose down       # stop
docker compose restart    # reload after editing nginx.conf
docker compose ps         # status
docker compose logs -f    # tail nginx logs
```

The `viewer` directory is bind mounted read-only as the web root, so edits to any file under it are live, no restart. Only `nginx.conf` needs one.

On localhost the viewer polls `files.json` and the open document every two seconds and repaints on a change. A deployed copy skips the poll and loads once.

## Deploy

If you want to deploy the flows publicly, just serve the `viewer` directory.

```nginx
server {
    root /path/to/app/dir/viewer;
    index index.html;

    add_header Cache-Control "no-store" always;

    location ~ \.md$ {
        default_type text/markdown;
    }
}
```

## License

MIT - see [LICENSE](LICENSE).

---

Built and maintained by Rob at [Websanova](https://www.websanova.com). I take freelance and contract work, including MVP projects built on the Starters. Check out the [hire page](https://www.websanova.com/hire) for more info.
