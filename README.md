# Starter Flows

Feature flow specs for the starter projects. Planning only, no application code.

Each feature gets one flow file: the logic, states, rules, edge cases, error handling, and the reasoning behind the choices, with mermaid diagrams. A flow is the source of truth. Code repos execute against it.

## Why

Flows are implementation-agnostic. A Stripe subscription flow is the same whether it lands in Laravel/Cashier, Rails, Django, Vue, React, or a mobile client - the third party behavior, the states, the webhook races, and the tax and promo rules do not change. Only the library choices do, and those are a code session problem.

Two reasons this repo exists:

* AI executes far better from an explicit spec than from a brainstorm prompt. Work the logic out once, then hand it over as a direct instruction.
* Flow docs and diagrams stay in one place instead of muddying each code repo.

## Workflow

1. Write or update the flow here until the logic is settled.
2. Open a session in the target code repo.
3. Paste the flow and give it one of these:
   * `Execute this flow.`
   * `Review this flow for missing parts to add to our code.`
   * `Review this flow against the code and report what needs updating.`

Any new feature or tweak goes through a flow first.

## Structure

Flows live in `flows/<feature>.md`, hyphenated. Every file follows the same section order - see [CLAUDE.md](CLAUDE.md) for the template.

Each flow carries a status:

* `draft` - still being worked out
* `approved` - logic settled, ready to execute
* `implemented` - executed in at least one repo

## Flows

| Flow | Status | Updated |
| ---- | ------ | ------- |
| -    | -      | -       |

## Projects

API

* [Starter Laravel API](https://github.com/websanova/starter-laravel-api)

APP

* [Starter Vue SPA](https://github.com/websanova/starter-vue-spa)
