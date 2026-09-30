# 010: The `goal/set` response arrives only when the woken turn terminates (observed on `echo`)

- **Status:** open question. It is potentially blocking for any synchronous goal controller
- **Area:** MSP goals / command acknowledgement timing
- **Observed on:** Muse Code 1.4.1 (1.4.1-R4503.1) and 1.4.0, `echo` provider,
  macOS arm64, 2026-09-30. **Not yet measured with a real model provider.**

## Summary

`GoalCommandResult` is documented as admission-only. On `echo`, however, the
response to a `goal/set` that wakes a goal turn is written only when that
turn terminates. By contrast, `turn/start` and `goal/clear` answer immediately.

Timeline (1.4.1, `goal/set` on an idle session):

```text
 162 ms  session/goalChanged  status=active
 389 ms  turn/started
 936 ms  RESPONSE goal/set         <- same instant as
 936 ms  turn/completed  (failed)
```

The durable log writes `runtime.command_intake.settled` before `turn/started`,
so the intake settles early but the response frame is flushed late. We see
the same pattern for busy `goal/set` and for `goal/resume` on a `blocked` goal.
`turn/start` answers in 72–94 ms, before its own `turn/started`.

## Impact on an embedding client

If this holds with a real model, where a goal turn can run for minutes, a
client that awaits the `goal/set` response blocks for the whole turn. Any
ack deadline shorter than the turn reports a false "lost ack".

## What would help

Confirm the intended behaviour. If `GoalCommandResult` is admission-only,
answer before the woken turn runs, as `turn/start` does. Otherwise document
that a waking goal command answers at turn termination.
