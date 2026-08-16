# Starter Flows

Feature flow specs for the starter projects.

| Flow | Status | Updated |
| ---- | ------ | ------- |
| -    | -      | -       |

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

Status: `draft` (being worked out), `approved` (ready to execute), `implemented` (executed in at least one repo). Section template in [CLAUDE.md](CLAUDE.md).

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
