---
description: How a flow file's Flow section is structured, and what phases mean
globs:
  - "public/specs/flows/**/*.md"
  - "public/specs/refs/**/*.md"
---

# Flow Structure

## Phases

- A feature split across numbered files is phases. `Create1Load.md` is phase 1, `Create2UserAction.md` is phase 2, `Create3Submit.md` is phase 3. Refer to them that way.
- `step` is loose and takes whatever is being counted, a state of a form component, a numbered line in `Flow`. It is not a defined term and never needs one.
- Content belongs to the phase that performs it. A phase never describes another phase's work, it links to it.

## Flow Section

- Every top level step names the system that owns it as the subject, `App` or `API`. One system per top level step, so a handoff is always a step boundary.
- First sentence names the action. Short, no lead-in. Reading only the first sentences is a full pass of the flow.
- Detail follows in the same bullet, never in place of the action.
- A bullet that is not an action gets cut, not reworded, unless it qualifies an action. Then it nests under that action at the third level, never as a sibling of the actions.
- A lone third level bullet is a smell. Roll it into its parent's trailing detail and nest only when several hang off the same action.
- A step describes only what happens in that step. Naming a later step's work is a defect. A forward reference is allowed only as the reason a decision is being made here.
- Detail cut from a step for length moves into that step's note. Nothing is deleted on the way.
- Renumbering steps renumbers the note titles that point at them.
- Top level steps line up with the diagram nodes, loosely. Not a hard rule.

## Links

- Links between spec files are viewer hash links, the manifest path minus `.md`. `[subscription guards flow](#flows/subscription/Guards)`. A relative `.md` link 404s.
