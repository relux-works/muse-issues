# 011: Identical goal adoption while `active` emits nothing and has no `noop` ack

- **Status:** open, medium (we handle it with read-after-write)
- **Area:** MSP goals / change gate
- **Observed on:** Muse Code 1.4.0 and 1.4.1, 2026-09-30

## Summary

Two `goal/set` with the same objective, sent 40–50 ms apart while that goal is
`active`: both are accepted, and exactly one `session/goalChanged` arrives.
This matches the documented change gate ("identical adoptions emit
nothing"). The `GoalCommandResult` of the second command, however, looks
exactly like a real change (`status: accepted`).

## Impact

A client that waits for a notification to confirm a re-apply of the goal the
server already holds (restore, reconcile, retry) waits forever. It has to add
a read to learn that nothing changed.

## What would help

Return a `noop` status in `GoalCommandResult` for an identical adoption, as
`CommandAckStatus` already does for compaction.
