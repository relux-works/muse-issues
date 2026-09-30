# 007: `serve --no-session-log` suppresses `session/goalChanged`

- **Status:** workaround
- **Area:** MSP notifications / serve flags
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1)

## Summary

With `muse serve --no-session-log`, `goal/set` succeeds, but the
`session/goalChanged` notification never arrives. Our client timed out
waiting for it. Without the flag (default durable session logging) the set and
clear events arrive as expected.

## Impact

A client that disables session logs for privacy or disk reasons silently
loses goal notifications. The two behaviours look unrelated, and nothing in
the flag's description links them.

## What would help

Either emit `goalChanged` independently of durable session logging, or
document that goal notifications require the durable log (and ideally refuse
`goal/*` with a clear error when it is disabled).
