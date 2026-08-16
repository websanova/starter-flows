---
description: Write exactly the flow changes that were agreed. No git.
argument-hint: [clarification, optional]
disable-model-invocation: true
allowed-tools: Read, Edit, Write, Grep, Glob
---

FLOW FILES ONLY. Write exactly what was discussed and agreed in a PREVIOUS message. Proposals made in the same response as this command are not authorized.

## Rules

- Do not add sections, cases, rules, or scope beyond what was agreed. Do not rewrite surrounding flow content. Do not create new files unless strictly necessary.
- Follow the flow file structure in CLAUDE.md exactly. Section order is fixed.
- Bump the `Updated:` header on any flow file touched.
- Update the README flows table when a flow is added or a status changes.

## Output format

After the edits, list changed files as relative markdown links, one line each, with a short phrase for what changed. Nothing else. No summary, no explanation of the approach, no next steps.

$ARGUMENTS
