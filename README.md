# Starter Flows

Feature flow specs and setup docs for the starter projects.

Part of the Starters, built at [Websanova](https://www.websanova.com).

## Docs

Full documentation at [websanova.com/docs/starter-api](https://www.websanova.com/docs/starter-flows).

- [Summary](docs/summary.md)

## Projects

| Project | Repo | Demo |
| ------- | ---- | ---- |
| Starter Flows | [starter-flows](https://github.com/websanova/starter-flows) | [flows](https://starter-flows.websanova.com) |
| Starter Laravel API | [starter-laravel-api](https://github.com/websanova/starter-laravel-api) | [api](https://starter-laravel-api.websanova.com) |
| Starter Vue SPA | [starter-vue-spa](https://github.com/websanova/starter-vue-spa) | [app](https://starter-vue-spa-app.websanova.com), [admin](https://starter-vue-spa-admin.websanova.com) |

## How It Works

Each feature gets one flow file in `public/specs/flows/`, holding a description, the shared terms, the requirements, the numbered steps, a mermaid diagram, notes and outstanding items. Anything that is not a feature gets a doc in `public/specs/docs/`, which has no fixed structure. Both are implementation-agnostic, so a Stripe subscription flow is the same in Laravel, Rails, Django, Vue, React, or mobile. Only the library choices differ, and those are a code session problem.

Every term used in either is defined once in [public/specs/docs/conventions/Terms.md](public/specs/docs/conventions/Terms.md) and copied word for word wherever it appears.

Work the flow out here first, then paste it into a session in the target code repo:

* `Execute this flow.`
* `Review this flow for missing parts to add to our code.`
* `Review this flow against the code and report what needs updating.`

Every status keyword is defined in [public/specs/docs/conventions/Status.md](public/specs/docs/conventions/Status.md). Section template in [CLAUDE.md](CLAUDE.md).

The viewer sidebar reads [public/files.json](public/files.json), a hand kept list of every file with its status and updated date. A new flow, doc or ref needs a line there as well as a row in [docs/summary.md](docs/summary.md).

## Docker

Markdown viewer for `public/specs/` at http://localhost:8088.

```bash
docker compose up -d      # start
docker compose down       # stop
docker compose restart    # reload after editing nginx.conf
docker compose ps         # status
docker compose logs -f    # tail nginx logs
```

The `public` directory is bind mounted read-only as the web root, so edits to any file under it are live, no restart. Only `nginx.conf` needs one.

On localhost the viewer polls `files.json` and the open document every two seconds and repaints on a change. A deployed copy skips the poll and loads once.

## Deploy

If you want to deploy the flows publicly, just serve the `public` directory.

```nginx
server {
    root /path/to/app/dir/public;
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
