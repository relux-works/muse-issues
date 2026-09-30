# 002: `session/read` does not return the goal state

- **Status:** blocking
- **Area:** MSP read model: `session/read`, `session/resume` snapshots
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64
- **Re-verified on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-09-30. `session/read` with `excludeItems` `true` and `false` returns `history.snapshot: null` and no goal in `session`. The schema note about `#22785 (E8)` is still present.

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

## Muse Code references (quoted from the exported 1.4.0 schema)

- `SnapshotState` ("the complete folded view at the snapshot cursor", tdd SS4.9.1): "One served site does not satisfy this type today, escalated under **#22785 (E8)**. The genesis snapshot rung serves `state: {"items": [...]}` alone … The fix is a serving change in the **#208/#14653** lane (serialize the `SessionViewState` the genesis path already folds, as the anchored path does)."
- `SessionHistory`, the history envelope shared by `session/resume`, `session/fork` and `session/read`: tdd SS2.5.2.
