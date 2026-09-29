# Status

Status: done
Updated: 2026-09-29

The `Status:` header line every flow, doc and ref carries. Listed in lifecycle order, `ref` last since it sits off the main line.

| Status | Applies to | Meaning |
| --- | --- | --- |
| wip | flow, doc, ref | Ideas being gathered. Not worked out yet. |
| ready | flow | Settled and ready to execute in a code repo. |
| built | flow | Built in at least one repo. |
| done | doc | The document is finished. It says nothing about whether what it describes is built. |
| ref | ref | A strategy that was evaluated and not taken. Kept for comparison and never built. |

A flow tracks the work, so `built` means the code exists. A doc tracks itself, so `done` means the writing is finished and nothing more. The two terminal words are separate to keep one from being read as the other.
