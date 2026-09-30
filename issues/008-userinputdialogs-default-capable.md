# 008: `userInputDialogs` absent means "capable", so headless clients receive must-answer requests

- **Status:** workaround
- **Area:** MSP capabilities / server-initiated requests
- **Observed on:** Muse Code 1.4.0 schema

## Summary

MSP 1.4 includes server-initiated requests the client must answer:
`approval/request` and `userInput/request`. The capability that governs user
input dialogs, `ClientCapabilities.userInputDialogs`, is documented so that
**absent means capable**. A minimal headless client that does not know about
the capability therefore opts in by default, and may receive requests it
cannot answer. That stalls or breaks the session.

## Impact

A trap for headless and automation clients. Our client refuses any
server-initiated request by contract (it treats one as a protocol error). With
the default capability, a single approval prompt can terminate the
connection.

## Workaround we use

We send `userInputDialogs: false` in `initialize` and start sessions in a
non-interactive approval mode.

## What would help

- Default to *not capable* when the field is absent (safer for `serve`), or
- document the default prominently next to `serve` and the approval modes.
