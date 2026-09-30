# 003: The goal state machine (`blocked`, `-32030`) is undocumented and asynchronous

- **Status:** workaround
- **Area:** MSP goals: `goal/set`, `goal/edit`, `goal/pause`, goal `status`
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64, `echo` provider

## Summary

The goal's `status` changes asynchronously after a `goal/set` is acknowledged,
and which goal verbs are accepted depends on that status. Neither is described
in the schema. The schema marks `status` as a verbatim free string.

What we observed:

1. After `goal/set`, the goal is acknowledged as `active`. If the goal turn then
   fails (on the offline `echo` provider it fails with `authRequired`; see
   [004](004-echo-provider-not-fully-offline.md)), the server moves the goal to
   **`blocked`** within about 1 s and emits
   `goalChanged{same objective, status: blocked}`.
2. In `blocked`, **`goal/edit` and `goal/pause` are rejected** with
   `-32030 invalid_goal_state`.
3. In `blocked`, **`goal/set` is accepted**, and `goal/set` is also accepted
   over `active`.
4. The `blocked` transition of the *previous* goal can arrive while the client's
   next goal command is in flight, interleaved with that command's own events.

## Impact on an embedding client

- A client that picks `goal/edit` for "change the existing goal", which is the
  natural reading of the verbs, fails intermittently depending on timing.
- A budget-enforcing client that sends `goal/pause` when a token ceiling is
  reached gets `-32030` in the transition window.
- Any client-side cache of the goal status is always potentially stale, so
  decisions keyed on it race the server. We hit that class four times before
  removing every status-based decision from our client.

## Workaround we use

We always use `goal/set` (never `goal/edit`) for apply and rollback. We send
`goal/pause` only when we last saw `active`, and treat `-32030` as a typed "not
applicable now" outcome.

## What would help

- Document the goal state machine: states, transitions, what triggers
  `blocked`, and which verbs are valid in which state (with the error code).
- Consider accepting `goal/edit` (and `goal/pause`) on `blocked`, or document
  `goal/set` as the always-valid replacement verb.
- Consider including the resulting state in `GoalCommandResult`.
