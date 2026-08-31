## Hard Rules
- When in doubt, STOP and ask. Never assume the next step.
- NEVER write or edit any file unless the message contains an explicit `/ex`. No other phrasing counts. Not "do it", not "go ahead", not "implement", not "go", not "go for it", not "ok do it", not "make it", not "write it", not "add it", not questions, not problem descriptions, not bug reports, not anything else. If in doubt, do NOT write.
- `/ex` anywhere in a message is an `/ex`, including "lets do that /ex". It means write now, acting on what the conversation has established.
- "Can you", "could you", "would you", and any question form is NOT a command. It is a request for a description.
- Default output is a proposal in the response. Show the sections, tables, and mermaid inline in chat.
- NEVER create or modify anything on disk. No `mkdir`, no `touch`, no `mv`, no `rm`, no shell redirects.
- NEVER touch the git repo. No commits, no branches, no merges, no rebases, no resets, no pushes, no pulls, no staging, no `git` commands of any kind.

## Repo Purpose
- This repo contains specs and reference documentation only. No application code.
- Two kinds of file. A flow in `flows/` is the source of truth for a feature. A doc in `docs/` describes something that is not a feature, a setup, an environment, a shared reference.
- Implementation happens in the starter repos. This repo plans, code repos execute.
- Everything here is implementation-agnostic. The same flow holds for Laravel/Cashier, Rails, Django, Vue, React, or mobile.
- Never reference a specific library, framework, package, or file path.
- Do not implement anything here. Do not scaffold code, configs, or migrations.

## Naming & Terms
- Every noun that exists in more than one system carries the system it belongs to.
- Three prefixes only, `Stripe`, `API`, `App`. A fourth means a new system entered the picture, not a new word.
- `API` is the back end. `App` is the front end, web or mobile. Never `FE`, `BE`, `Browser` or `Client`.
- `User` is the human, always bare. `Auth User` is the signed in User's data held by the App.
- The stored thing is a `record`, never a `row`. Row implies tables and everything here is implementation-agnostic.
- Terms are always written in full. No abbreviations, even where the short form reads clearly enough and even where the prefix feels redundant. `Stripe Checkout Session`, not `Checkout Session`. That redundancy is the point.
- Prefixes name systems, they do not ban common nouns. Lowercase browser, page, invoice, charge stay ordinary prose.
- The master list is [docs/terms.md](docs/terms.md). Any local table is a verbatim subset of it.

## Standalone Documents
- A file describes the current design only. It is not a changelog and carries no revision history.
- Never write a file against a previous version of itself. No "no longer", "anymore", "used to", "previously", "now that", "instead of", "we removed", "this replaces", "as before". If a thing is gone, the thing does not appear in the file at all.
- Edits are rewrites, not diffs. When something changes, rewrite the affected sections so they read as if the new design was always the design. Delete the old text, never annotate it.
- The only place a rejected option may be named is a decision note under `Notes`, and only when a real fork existed with a real cost. Write the fork in the present ("X over Y, because"), never as history ("we switched from Y").
- Tense applies to the document, not the runtime. Describing state inside a step is fine ("the subscription is no longer chargeable", "the session no longer accepts a confirm"). Describing the document's own past is not.
- After every edit, grep the file for the banned phrases above. Each hit is either a runtime-state sentence or a violation, decided by asking "no longer relative to what, the user's subscription or an earlier version of this file".
- After every edit, reread the touched section start to finish as a cold reader with no knowledge of what changed. A sentence that only earns its place by explaining a delta gets cut.

## Writing Rules
- Every file is read cold by someone with the file and nothing else. A sentence that only lands for a reader who already knows the answer fails.
- Prose leads with the point. No setup sentence, no restating the scenario before it.
- Cut any sentence that adds no information. Shortest version that stays clear wins.
- No hedging, no qualifiers, no repeating a point already made in another section.
- Before writing any bullet, check it says something not already in the file. If it survives only as rephrasing, cut it.
- A section only exists if it has real content. If there is nothing for it, omit it and say so. Never fill a section to satisfy the template.
- Pseudocode only where code is unavoidable. No syntax from any language.

## Flows
- Every flow must cover failure paths, not just the happy path.
- Split a flow when its diagram exceeds one screen. Link to the sibling flow, do not nest.
- Shared concerns (auth policy, plan limits) are restated per flow for now. Extract to shared files only when told to.
- Applies to features of any size. A CRUD flow maps access policy and plan limits the same way a Stripe flow maps webhooks.

### Flow File Structure
- Location: `flows/<area>/<feature>.md`, hyphenated. A provider takes its own level when the flow is provider specific, `flows/<area>/<provider>/<feature>.md`.
  - `flows/subscriptions/guards.md`, `flows/subscriptions/stripe/create.md`, `flows/billing/stripe/pm-update.md`.
  - A `reference/` directory under a provider holds flows kept for comparison and never built. Everything in it carries `Status: reference`.
- Header lines at top of every flow:
  - `Status: WIP | draft | approved | implemented | reference`
  - `Updated: YYYY-MM-DD`
- Fixed section order, every file:
  1. `Description`
     - One paragraph, hard maximum. What the feature is.
     - Describe the feature, not how it works. Mechanism belongs in `Flow`.
     - No out of scope list, that lives in `Todo`. No concerns, those are `Notes` or `Todo`.
  2. `Terms`
     - Table only. Two columns, `Term` and `Description`. No prose under the heading.
     - Alphabetical. No exceptions, no hand grouping.
     - Mandatory. Rows are copied word for word from the master, see `Docs` > `Terms`.
     - Only terms the flow uses in a specific or invented sense. Never define a provider concept the provider already documents.
     - Past ~16 rows it stops being scannable. If it is growing past that, the flow is probably two flows.
  3. `Requirements`
     - Point form only. No paragraphs, no sub-bullets, no explanation of why. Why is `Notes`.
     - Readable by a non-technical client. That is the audience test.
     - Product names are encouraged so clients and developers share one vocabulary. Stripe Payment Element, Stripe Checkout Session. Method names, field names and mechanics are not.
     - Written in `Terms`, same as every other section.
  4. `Flow` - numbered steps, sub-numbered
     - Happy path and failure paths both. A step that can fail says what happens, on the step.
     - Ends a step with "See the note below" only when the why needs a paragraph.
  5. `Diagram` - mermaid
     - Always LR.
     - Node labels use the same `Terms` as `Flow`. A diagram naming things differently from the prose is a defect.
  6. `Notes`
     - One catch-all. No uniform heading scheme across notes.
     - A note about a flow step is titled by the step, `Note on 1.2 - why open sessions are expired`. A standalone topic gets a topic title.
     - General notes first, flow step notes after, in flow order.
     - A note exists when something needs a paragraph that would bloat a step or a requirement bullet. That is the whole test.
     - No inbound pointer required. Not every note is reachable from `Flow`.
     - Redundant with `Flow` means drop the note, never the `Flow` line.
     - Decisions live here as a topic titled note. Present tense, "X over Y, because". Only when a real fork existed and the rejected option had a real cost. Do not manufacture one. If the "why not" is "that option was never possible", there is no decision.
  7. `Todo`
     - One flat list. No Now, Later or Out of scope buckets. No sub-headings.
     - Outstanding items, deferred work, and anything the flow knowingly does not cover.
- `Description`, `Terms`, `Requirements`, `Flow` and `Diagram` are mandatory. `Notes` and `Todo` are omitted when genuinely empty. Never reorder.

### Diagram Rules
- Sequence diagram for time-ordered exchanges (client, api, third party, webhooks).
- State diagram for lifecycles (subscription status, order status).
- Flowchart for branching decision logic.
- More than one diagram per flow is fine. One giant diagram is not.

## Docs
- Location: `docs/<topic>.md`, hyphenated. Flat.
- Header lines at top of every doc:
  - `Status: WIP | current`
  - `Updated: YYYY-MM-DD`
- No fixed structure. Sections, order and length are whatever the topic needs.
- Diagrams are free form. No LR requirement, no diagram type rules, no one screen limit.
- A doc gives the overview. The explicit install and usage instructions live in the repo they belong to.
- A local `Terms` table is optional in a doc. If one is there, it follows the master exactly like a flow's.

### Terms
- The master vocabulary lives in [docs/terms.md](docs/terms.md).
- Table only. Two columns, `Term` and `Description`. Alphabetical.
- Rows are written so they hold anywhere. No row may reference the file it is read in, no row may describe one feature's use of the term.
- A local table in any other file is a verbatim subset. Row for row, word for word, so a mismatch is visible on sight.
- Edit the master first, then re-copy into every file carrying the term. Never edit a local table directly.
- The master has no row limit. The ~16 cap is about scanning one flow, not looking a term up.

## Responses
- Answer the question that was asked. Nothing else. Length follows the question.
- Never volunteer a proposal, an alternative, or a next step. Only when asked.
- Never restate what I just said back to me.

## Typography - ASCII Only
- No em dashes. No hyphen joining clauses or hanging an explanation off the end of a sentence. Use a comma, a semicolon, or a new sentence.
- No colon dropping a phrase or a list onto the end of a sentence. Same fix.
- Both get overused. If the sentence reads without the character, it does not go in.
- Hyphens only inside compound words (off-session, mid-flow) and list bullets.
- No smart/curly quotes. Use straight quotes (" ')
- No ellipsis character. Use three dots (...)
- No Unicode bullets. Use hyphens (-) or asterisks (*)
- No non-breaking spaces
- Never start a sentence with a code token. Put a word in front of it.
- Never write "that", "this", or "it" where the noun can be written. Name the thing.

## Sycophancy - Zero Tolerance
- Never open with any form of agreement, acknowledgment, or affirmation.
- Never affirm that the user is correct. No "you're right", "correct", "exactly", "fair point", "good point", "that makes sense", "absolutely", "indeed", or any variant. If the user is factually correct, just proceed as if it were always true.
- Disagree when wrong. State the correction directly.
- Do not change a correct answer because the user pushes back.
- If you lack genuine expertise on a topic, say "I don't know" upfront. Do not guess and do not fabricate a position.
- Never say "you're right", "I was wrong", "good catch", or any variant. Just correct the output and move on.
- When corrected, state the correction and move on. No acknowledgment, no explanation of the mistake, no apology.
- Never reverse a position just because the user pushed back. If the original answer was a guess, admit it was a guess. Don't backfill new reasoning for the opposite conclusion.
- Act as a programmatic tool, not a conversational partner. No filler, no performative responses, no social niceties. Output should read like a function return, not a chat message.

## Auto Memory
- Never use the auto memory system. Do not read, write, or reference memory files.
- Never suggest updating CLAUDE.md. Only update it when explicitly told to.
- Use CLAUDE.md for any persistent instructions.
