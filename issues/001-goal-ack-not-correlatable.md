# 001: A goal command's effect cannot be correlated to the command

- **Status:** open, medium. A client-side workaround exists (read-after-write plus replay); see Correction
- **Area:** MSP goals: `goal/set`, `goal/clear`, `session/goalChanged`
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64, `echo` provider
- **Re-verified on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-09-30. `GoalCommandResult` and `session/goalChanged` are unchanged in the 1.4.1 schema, and `goal/set` still answers `{commandId, status, turnId}` only.

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

The wire contract permits this sequence. Our fake server reproduces it
deterministically. We have **not** observed the real server emitting such a
stale event: a replaced goal appears to stop emitting. Our earlier wording
("reproduced on the real 1.4.0 binary") was overstated, and we corrected it on
2026-09-30.

## Impact on an embedding client

A controller that must *know* which goal the session is working on (for
example to report state to a user, enforce budgets per goal, or roll back a
bad goal) cannot guarantee correctness on rollback. That operation is exactly
where correctness matters most. Our host has to either accept a documented
race or disable rollback to a previously used objective.

## Correction (2026-09-30): what does work today

- **Read-after-write.** `session/resume {history:"snapshot"}` on the loaded
  session returns `SnapshotState.goal` on 1.4.0 and 1.4.1 (see
  [002](archive/002-session-read-has-no-goal.md)). A client can confirm a goal command
  by reading after the response.
- **Durable correlation exists.** `session/goalChanged.sourceRange.first.id`
  names the `session.goal_control.applied` record, and that record carries the
  client's `commandId` (`causation_id`, `record.command_id`) and the goal's
  `goal_id` and `revision`. Only the wire `Goal` block lacks them.
- **Replay is idempotent.** Re-sending `goal/set` with the same `commandId` and
  payload returns the same result and emits nothing. The same id with a
  different payload is `-32030 command_id_conflict`. This is the correct
  recovery for a lost response.

## Workarounds we tried

- Matching by objective identity (the requested objective is the ack, the
  previous one is lifecycle noise) fails when the rollback target equals an
  earlier objective.
- Read-after-write via `session/read` is impossible, because the goal is not
  in the read result (see [002](archive/002-session-read-has-no-goal.md)).
- Using the cached state in the client makes things worse; every variant
  diverged from the server under interleaving.

## What would still help (in order)

1. Put the goal identity on the wire: `Goal.goalId` and `Goal.revision`
   (both already durable in the goal store), so no client needs a second
   request or the session log.
2. Put the originating `commandId` on `session/goalChanged` when a client
   command caused it.
3. Document that `GoalCommandResult` is admission-only. We measured it with a
   real provider: the response comes about 0.6 s after the request, long
   before the woken turn ends (see [010](archive/010-goal-set-response-waits-for-woken-turn.md)).

(We dropped the earlier "put a `viewCursor` in the result" ask: cursors are
opaque and clients must not compare them.)

## Muse Code references (quoted from the exported 1.4.0 schema)

- `goal/set`, `goal/clear`, `goal/edit`, `goal/pause`, `goal/resume`: "tdd SS3.18, spec 14408; enrolled by **#33066**". `GoalCommandResult` is "the shared SS3.18 goal ack, admission-only".
- `session/goalChanged`: "tdd SS4.6.2"; `viewCursor` is "opaque, strictly monotonic (tdd SS4.1)"; "identical adoptions emit nothing (the change gate)".
