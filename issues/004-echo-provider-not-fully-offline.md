# 004: The offline `echo` provider refuses turns (`authRequired`) and emits no `tokenUsage`

- **Status:** open, medium–high (the usage path cannot be tested offline)
- **Area:** testing / offline provider
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64
- **Re-verified on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-09-30. Every turn ends `terminal: failed`, `error.kind: authRequired` (`not logged in: run /login to add an API key`, `retryable: false`), and no `session/tokenUsage` is emitted.

## Summary

The built-in `echo` provider is the natural choice for hermetic tests (no
network, no credentials). With it, `initialize`, `session/start` and the goal
verbs work, but:

1. A turn (for example `turn/start` with `ifBusy: queue`) completes with
   `turn/completed` of kind **`authRequired`**, message
   `not logged in: run /login to add an API key`, `retryable: false`.
2. No **`session/tokenUsage`** notifications are emitted, and `turn/completed`
   carries no `usage`. Over 10 consecutive
   runs the cumulative usage stayed at 0.

## Impact on an embedding client

- Turn delivery (queued notices, steering) cannot be exercised end to end
  offline.
- Token-budget enforcement driven by `session/tokenUsage` cannot be tested
  against the real server at all. We can only test it against our own fake.
- Because the goal turn fails, the goal moves to `blocked`
  ([003](003-goal-state-machine-undocumented.md)). Offline tests therefore
  always run in the error path of the goal state machine, not the happy path.

Also, in a fresh private home, `model/list` returns 0 rows
(`source: bundledCatalog`).

## What would help

- The schema's `ModelCatalogSource` already knows a `fakeCatalog`. Exposing
  an internal fake provider/model to external clients, with deterministic
  replies, synthetic usage and a configurable turn duration, would cover this
  issue and let clients measure [010](archive/010-goal-set-response-waits-for-woken-turn.md)
  offline.

- `echo` turns that complete successfully without login (they echo the input).
- Synthetic, deterministic `session/tokenUsage` from `echo` (for example
  tokens = input length), so budget logic is testable offline.
