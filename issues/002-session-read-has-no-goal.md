# 002: `session/read` does not return the goal state

- **Status:** blocking
- **Area:** MSP read model: `session/read`, `session/resume` snapshots
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64

## Summary

The schema defines `SnapshotState.goal` (the full folded view, including the
goal), but no read request we can make returns it:

- `session/read` with `excludeItems: true` returns
  `history: {"mode":"none","items":null,"snapshot":null,"noneReason":"excluded"}`.
  The `session` object has no goal field.
- `session/read` with `excludeItems: false` returns `history.mode: "inline"`
  with transcript items and `history.snapshot: null`. There is still no
  structured goal.
- `session/read` has no history-mode selector, so there is no way to ask it
  for a snapshot.
- The route through `session/resume` with a snapshot is documented in the
  exported schema itself as not serving the full state on the genesis rung:
  "The genesis snapshot rung serves `state: {"items": [...]}` alone … while
  the anchored rung serves the full folded state", escalated under an internal
  reference `#22785 (E8)`. We do not have access to that tracker and only
  quote the schema text.

## Reproduction (raw)

After `goal/set` on an isolated `echo` session:

```text
request:  {"sessionId":"<id>","excludeItems":true}
response: {"session":{"sessionId":"<id>","path":"…/session.jsonl","status":"running",…},
           "viewCursor":"v:<id>:6",
           "history":{"mode":"none","items":null,"snapshot":null,"noneReason":"excluded"}}

request:  {"sessionId":"<id>","excludeItems":false}
response: {"session":{…,"status":"idle",…}, "viewCursor":"…",
           "history":{"mode":"inline","items":[…],"snapshot":null}}
```

Neither response contains the goal.

## Impact on an embedding client

There is no authoritative way to ask the server "what is the goal right now?".
Combined with [001](001-goal-ack-not-correlatable.md), a client cannot confirm
the effect of its own goal commands. Transcript items are not a substitute:
they are presentation, not state.

## What would unblock us

- `session/read` returns the goal block (the `SnapshotState.goal` shape) and
  the cursor it is folded at, or
- a dedicated read-only request such as `goal/get` → `{goal | null,
  viewCursor}`, or
- the genesis snapshot rung serving the full `SnapshotState`, so that
  `session/resume` with a snapshot works for this purpose.
