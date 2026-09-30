# 001: A goal command's effect cannot be correlated to the command

- **Status:** blocking
- **Area:** MSP goals: `goal/set`, `goal/clear`, `session/goalChanged`
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64, `echo` provider

## Summary

After a client sends `goal/set`, there is no way to know which
`session/goalChanged` notification (if any) reflects *that* command:

- `goal/set` returns `GoalCommandResult {commandId, status, turnId?}`, which is
  **admission-only**. It carries no view cursor and no durable record id.
- `session/goalChanged` carries the goal block (`objective`, `status`,
  `percentComplete`), `sourceRange` and a monotonic `viewCursor`, but **nothing
  that ties it to a command** (no `commandId`).

A client therefore has to infer the acknowledgement from the goal content, and
that inference is ambiguous.

## The failing case

1. Set goal A. The server acknowledges it; A becomes `active`.
2. Set goal B. It is acknowledged.
3. Roll back: set goal A again.
4. Meanwhile, the server emits a *late* lifecycle event for A from before
   step 3 (for example A → `blocked`; see [003](003-goal-state-machine-undocumented.md)).
   It carries objective A.
5. The client receives `goalChanged{objective: A, status: blocked}` and cannot
   distinguish "the rollback to A took effect" from "a stale event of the old
   A".

If the client treats it as the ack, it reports success while the server still
holds B. If it waits for a "fresh" A, it cannot tell whether one will come,
because an identical adoption emits nothing (the change gate in the schema).

We reproduced this deterministically: a fake server at 3/3, and the real
1.4.0 binary across delay sweeps of 0–800 ms between commands.

## Impact on an embedding client

A controller that must *know* which goal the session is working on (for
example to report state to a user, enforce budgets per goal, or roll back a
bad goal) cannot guarantee correctness on rollback. That operation is exactly
where correctness matters most. Our host has to either accept a documented
race or disable rollback to a previously used objective.

## Workarounds we tried

- Matching by objective identity (the requested objective is the ack, the
  previous one is lifecycle noise) fails when the rollback target equals an
  earlier objective.
- Read-after-write via `session/read` is impossible, because the goal is not
  in the read result (see [002](002-session-read-has-no-goal.md)).
- Using the cached state in the client makes things worse; every variant
  diverged from the server under interleaving.

## What would unblock us (any one)

1. Put the resulting **`viewCursor`** (or the durable record id of the goal
   record) in `GoalCommandResult`. The client then acks on the first
   `goalChanged` whose `viewCursor >=` that value.
2. Put the originating **`commandId`** on `session/goalChanged` (or in its
   `sourceRange`) when a client command caused the change.
3. Make `session/read` return the authoritative goal plus cursor
   ([002](002-session-read-has-no-goal.md)), so a client can confirm by
   reading.

## Muse references (quoted from the exported 1.4.0 schema)

- `goal/set`, `goal/clear`, `goal/edit`, `goal/pause`, `goal/resume`: "tdd SS3.18, spec 14408; enrolled by **#33066**". `GoalCommandResult` is "the shared SS3.18 goal ack, admission-only".
- `session/goalChanged`: "tdd SS4.6.2"; `viewCursor` is "opaque, strictly monotonic (tdd SS4.1)"; "identical adoptions emit nothing (the change gate)".
