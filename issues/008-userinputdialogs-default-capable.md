# 008: `userInputDialogs` absent means "capable", so headless clients receive must-answer requests

- **Status:** open, high
- **Area:** MSP capabilities / server-initiated requests
- **Observed on:** Muse Code 1.4.0 schema
- **Re-verified on:** Muse Code 1.4.1 (1.4.1-R4503.1) schema, 2026-09-30. `ClientCapabilities` is unchanged.

## Summary

MSP 1.4 includes server-initiated requests the client must answer:
`approval/request` and `userInput/request`. The capability that governs user
input dialogs, `ClientCapabilities.userInputDialogs`, is documented so that
**absent means capable**. A minimal headless client that does not know about
the capability therefore opts in by default, and may receive requests it
cannot answer. That stalls or breaks the session.

## Addendum (2026-09-30)

- `userInputDialogs: false` withholds only `userInput/request`. **Nothing
  withholds `approval/request`**, which is "presented to every subscribed
  connection".
- `session/resume` re-issues pending server-initiated requests right after its
  response, so a client that reads state through resume receives them again.

## Impact

A trap for headless and automation clients. Our client refuses any
server-initiated request by contract (it treats one as a protocol error). With
the default capability, a single approval prompt can terminate the
connection.

## Workaround we use

We send `userInputDialogs: false` in `initialize` and start sessions in a
non-interactive approval mode.

## What would help

- A client capability declaring "no interactive surfaces", under which the host
  applies the session approval mode's default (deny or allow) instead of
  parking a prompt.
- Default `userInputDialogs` to *not capable* when absent, or document the
  default prominently next to `serve`.
- Document `denyUnmatched` semantics with an empty policy.

## Muse Code references (quoted from the exported 1.4.0 schema)

- `approval/request` and `userInput/request`: "Server-initiated must-answer request … (SS5.3; indexed by **#23840**)".
