# Muse issues from a headless MSP integration

This repository collects problems we found in **Muse** while embedding
`muse serve` as a headless backend. An external controller drives it over MSP:
it starts sessions, sets goals, rolls goals back, sends turns, and enforces a
token budget.

Each issue says what we observed, on which build, how to reproduce it, what it
blocks for a client like ours, the workaround we use today, and what would
unblock us. We keep this list up to date as we learn more. Corrections are
welcome: if an item is a misuse on our side, tell us and we will close it.

## Environment

- **Build:** `Muse Code 1.4.0 (1.4.0-R4302.1)`, serverInfo
  `muse 1.4.0`, userAgent build `aebe0c188b3832bf0e2f68a30f797b42e7930072`.
- **Platform:** macOS, arm64.
- **Pinning:** the binary is a pinned copy (SHA-256
  `b7e1ebdf6b5c7e67ddb8ff4b917461f555758545c7db25076b3696d84018d752`), run
  with `MUSE_NO_AUTO_UPDATE=1`.
- **Isolation:** every run uses a fresh `HOME` and fresh XDG directories.
- **Provider:** tests use the built-in offline `echo` provider unless stated
  otherwise.
- **Schema:** MSP schema exported with
  `muse schema generate-json-schema` (schema version 1, `experimental: false`).
  The client-facing shapes we use are identical between 1.3.0 and 1.4.0.

## About Muse issue references

We have no access to the Muse team's tracker. Every `#NNNNN` or `tdd SS…`
reference in these files is quoted from the descriptions in the MSP schema
exported from the 1.4.0 binary, so it can be matched to internal work items.

## Index

| # | Title | Area | Impact on us |
| --- | --- | --- | --- |
| [001](issues/001-goal-ack-not-correlatable.md) | A goal command's effect cannot be correlated to the command | MSP goals | **Blocking.** We cannot tell reliably that a rollback took effect. |
| [002](issues/002-session-read-has-no-goal.md) | `session/read` does not return the goal state | MSP read model | **Blocking.** There is no authoritative read-after-write for goals. |
| [003](issues/003-goal-state-machine-undocumented.md) | The goal state machine (`blocked`, `-32030`) is undocumented and asynchronous | MSP goals | High. Our client broke several times before we reverse-engineered it. |
| [004](issues/004-echo-provider-not-fully-offline.md) | The offline `echo` provider refuses turns (`authRequired`) and emits no `tokenUsage` | Testing | Medium. Turns and token budgets cannot be tested offline. |
| [005](issues/005-macos-keychain-blocks-headless-serve.md) | On macOS an auth file makes `muse serve` hang on a Keychain UI prompt | Headless / macOS | Medium. Headless and CI use with credentials is impossible. |
| [006](issues/006-clientinfo-name-regex-not-in-schema.md) | The `clientInfo.name` pattern is enforced at runtime but missing from the schema | Schema | Low. It only cost debugging time. |
| [007](issues/007-no-session-log-suppresses-goalchanged.md) | `serve --no-session-log` suppresses `session/goalChanged` | MSP notifications | Low–medium. It is a hidden coupling. |
| [008](issues/008-userinputdialogs-default-capable.md) | `userInputDialogs` absent means "capable", so headless clients receive must-answer requests | MSP capabilities | Medium. It is a trap for headless clients. |
| [009](issues/009-silent-auto-update-changes-protocol-surface.md) | The launcher auto-updates silently under a running integration | Distribution | Medium. Reproducibility suffers. |

## Status legend

- `open`: observed and unresolved.
- `workaround`: we have a client-side workaround that costs us something.
- `blocking`: no client-side fix is possible; it needs a Muse change.

Every issue file carries its status in its header.
