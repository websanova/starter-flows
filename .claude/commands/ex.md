---
description: Write exactly what was agreed. No git.
argument-hint: [clarification, optional]
disable-model-invocation: true
allowed-tools: Read, Edit, Write, Grep, Glob
---

SPEC FILES ONLY. Write exactly what was discussed and agreed in a PREVIOUS message. Proposals made in the same response as this command are not authorized. If `$ARGUMENTS` is present it directs what to write.

- Do not add sections, cases, rules, or scope beyond what was agreed. Do not rewrite surrounding content. Do not create new files unless strictly necessary.
- Bump the `Updated:` header on any file touched, then match the date in the README table and `viewer/files.json`. A new file or a status change needs a row in both.
- After the edits, list changed files as relative markdown links, one line each, with a short phrase for what changed.
- Nothing else. No summary, no explanation of the approach, no next steps.

$ARGUMENTS
