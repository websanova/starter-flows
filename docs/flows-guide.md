# Flows Guide

Each feature gets one flow file in `public/specs/flows/`, holding a description, the shared terms, the requirements, the numbered steps, a mermaid diagram, notes and outstanding items. Anything that is not a feature gets a doc in `public/specs/docs/`, which has no fixed structure. A strategy that was evaluated and not taken gets a ref in `public/specs/refs/`, kept for comparison and never built.

All three are implementation-agnostic, so a Stripe subscription flow is the same in Laravel, Rails, Django, Vue, React, or mobile. Only the library choices differ, and those are a code session problem.

Every term used is defined once in [Terms](../public/specs/docs/conventions/Terms.md) and copied word for word wherever it appears. Every status keyword is defined in [Status](../public/specs/docs/conventions/Status.md). The section template for a flow lives in [CLAUDE.md](../CLAUDE.md).

## Adding a file

A new flow, doc or ref needs two entries by hand.

- A line in [public/files.json](../public/files.json), which the viewer sidebar reads for the status and updated date.
- A row in [Summary](summary.md).

## Using a flow in a code repo

Work the flow out here first, then paste it into a session in the target code repo.

* `Execute this flow.`
* `Review this flow for missing parts to add to our code.`
* `Review this flow against the code and report what needs updating.`

The starter repos each carry a `/flow` command that reads these files from a local clone. Point `CLAUDE.local.md` in the code repo at your clone, then `/flow subscription stripe create` compares the flow against that repo and reports what is built, what is missing, and what the other side expects it to expose.
