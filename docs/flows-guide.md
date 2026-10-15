# Flows Guide

There are no set rules on how to structure the flows and docs, the repo only serves as a starting point. It was made as a standalone viewer precisely so that it can be easily modified to suit specific needs. If you want a new tab/section like "archive" just get the agent to add it in.

There is a predefined structure for the flows in the agents ([CLAUDE.md](../CLAUDE.md)) file to get started with as well but that can also be changed as required.

Some basic recommendations to follow for organizing the flows and docs:

* Split flows up into manageable parts if they get too long or bloated. For example the subscription stripe create has 3 parts ([Load](../public/specs/flows/subscription/stripe/Create1Load.md), [User Action](../public/specs/flows/subscription/stripe/Create2UserAction.md), [Submit](../public/specs/flows/subscription/stripe/Create3Submit.md)), each of which can be tackled and refined separately.
* Don't mix concepts with implementation. For example the [MCP Client Login](../public/specs/flows/mcp/ClientLogin.md) flow strictly describes the mechanics of how to setup the MCP with a separate SPA and API with no actual implementation details. In this case the details were worked out in the process of implementation and then later documented under [Laravel MCP](../public/specs/docs/laravel/Mcp.md) as more of a "this is how it was implemented and why". It's not a flow directly.
* Don't get stuck on making the docs and flows perfectly organized. Just throw a file in somewhere and reorg with the agents when something better comes to mind.

Every term is defined once, in the master list [Terms](../public/specs/docs/conventions/Terms.md), and never repeated in another file. The viewer underlines each term wherever it appears, and the underline shows the description from the master list and links back to it. Every status keyword is defined in [Status](../public/specs/docs/conventions/Status.md). The section template for a flow lives in [CLAUDE.md](../CLAUDE.md), with the phase and layering rules in [flow-structure](../.claude/rules/flow-structure.md).

## Adding a file

Drop the file into whichever section fits, flows, docs, refs or one you added, and add a line for it in [public/files.json](../public/files.json), which the viewer sidebar reads. The line carries the path, label, status and updated date. The parts of a split flow share a `group` key holding the flow name, and each label is just the part.

A new noun that exists in more than one system also needs a row in [Terms](../public/specs/docs/conventions/Terms.md).

## Implementing flows

This may vary per agent, but the workflow is fundamentally the same. First we'll need the repos agent to know where the flows live. With Claude this can be done by making an entry in `CLAUDE.local.md`. The repo docs themselves contain more details on how to set this up, see [Starter Laravel API](https://github.com/websanova/starter-laravel-api) and [Starter Vue SPA](https://github.com/websanova/starter-vue-spa).

The starter repos also include a `/flow` command which if you're using Claude allows you to just fire off a flow directly with something like `/flow subscription stripe create`.

You'll want to work flows out here first for the most part. However, sometimes it's simpler to tweak them from the repo as you are implementing. Make sure to just ask the agent to do a clean up from within a flows repo session to make sure it picks up all the local rules for formatting and referencing terms which a source code repo would not know about.
