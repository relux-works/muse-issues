# 007: `serve --no-session-log` suppresses `session/goalChanged` (by design; docs gap)

- **Status:** reframed, low. Documentation gap, not a bug
- **Area:** MSP notifications / serve flags
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1)
- **Re-verified on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-09-30. With `--no-session-log`, `goal/set` is accepted but NO `session/goalChanged`, turn or status notification arrives; only `session/started` does. `serve --help` describes the flag only as "Use memory-only sessions".

## Correction (2026-09-30)

This is the schema's **ephemeral profile**, and `initialize` announces it:
- `sessionDurability: "ephemeral"`;
- `Session.path` is empty;
- `session/resume`, `session/read` and `view/page` answer `-32601`.

A client can detect it up front. What is missing is documentation: `serve
--help` says only "Use memory-only sessions", with no mention that the whole
read and notification plane is withheld. Our "hidden coupling" framing was
too strong.

## Original report

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
