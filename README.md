# Muse Code issues from a headless MSP integration

This repository collects problems we found in **Muse Code** while embedding
`muse serve` as a headless backend. An external controller drives it over MSP:
it starts sessions, sets goals, rolls goals back, sends turns, and enforces a
token budget.

Each issue says what we observed, on which build, how to reproduce it, what it
blocks for a client like ours, the workaround we use today, and what would
unblock us. We keep this list up to date as we learn more. Corrections are
welcome: if an item is a misuse on our side, tell us and we will close it.

## Environment

- **Originally observed on:** `Muse Code 1.4.0 (1.4.0-R4302.1)`, serverInfo
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

## Version check (2026-09-30)

- The latest release on BOTH launcher channels, `muse-stable` and
  `muse-canary`, is **1.4.1-R4503.1**; these are the only channels the launcher
  accepts. No newer build is available.
- The issues were re-verified on 1.4.1 where it was safe to do so (see each
  file). The 1.4.1 schema is identical to 1.4.0 on every surface these issues
  touch.
- An independent second review (2026-09-30) corrected 002 (the goal IS
  readable via the resume snapshot), narrowed 001 and 009, reframed 007, and
  added 010–014. 006 was withdrawn: it was our error.

## About Muse Code issue references

We have no access to the Muse Code team's tracker. Every `#NNNNN` or `tdd SS…`
reference in these files is quoted from the descriptions in the MSP schema
exported from the 1.4.0 binary, so it can be matched to internal work items.

## Index

| # | Title | Area | Muse Code refs | Impact on us |
| --- | --- | --- | --- | --- |
| [001](issues/001-goal-ack-not-correlatable.md) | A goal command's effect is not correlated to the command on the wire | MSP goals | #33066 · SS3.18, SS4.6.2 | High. A client-side workaround exists (a read plus the durable log); we want the goal id on the wire. |
| [002](issues/002-session-read-has-no-goal.md) | `session/read` has no goal (**corrected:** resume-snapshot does return it) | MSP read model | #22785 (E8) · #208/#14653 · SS4.9.1 | Low. We ask for a lease-free goal read and for the stale E8 note to be clarified. |
| [003](issues/003-goal-state-machine-undocumented.md) | The goal state machine (`blocked`, `-32030`) is undocumented and asynchronous | MSP goals | #33066 · SS3.18, SS4.6.2 | Medium. Worked around. |
| [004](issues/004-echo-provider-not-fully-offline.md) | The offline `echo` provider refuses turns (`authRequired`) and emits no usage | Testing | — | Medium–high. The budget path is untestable offline. |
| [005](issues/005-macos-keychain-blocks-headless-serve.md) | On macOS an auth file makes `muse serve` hang on a Keychain UI prompt | Headless / macOS | — | **High.** It blocks headless real-model use on macOS. |
| [006](issues/006-clientinfo-name-regex-not-in-schema.md) | ~~The `clientInfo.name` pattern is enforced at runtime but missing from the schema~~ | Schema | — | **Withdrawn.** Our error; the schema has the pattern. |
| [007](issues/007-no-session-log-suppresses-goalchanged.md) | `serve --no-session-log` suppresses `goalChanged` (by design, ephemeral profile) | Docs | — | Low. A documentation gap. |
| [008](issues/008-userinputdialogs-default-capable.md) | Must-answer server requests reach headless clients (`approval/request` has no opt-out) | MSP capabilities | #23840 · SS5.3 | High. |
| [009](issues/009-silent-auto-update-changes-protocol-surface.md) | The launcher auto-updates silently under a running integration | Distribution | — | Low. Pinning works, and `schema.fingerprint` already exists. |
| [010](issues/010-goal-set-response-waits-for-woken-turn.md) | The `goal/set` response arrives only when the woken turn terminates (on `echo` only) | MSP goals | SS3.18 | **Resolved.** Not reproduced with a real provider (response in ~0.6 s). |
| [011](issues/011-identical-goal-adoption-has-no-noop-ack.md) | Identical goal adoption emits nothing and has no `noop` ack | MSP goals | SS4.6.2 | Medium. |
| [012](issues/012-subagent-usage-not-in-session-cumulative.md) | Subagent/workflow usage is excluded from the session cumulative | Usage | — | High for budget enforcement. |
| [013](issues/013-serve-lacks-no-foreign-personal-context.md) | `serve` lacks `--no-foreign-personal-context` (only `exec` has it) | Isolation | — | Medium–high. |
| [014](issues/014-resume-does-not-rewake-unfinished-goal.md) | What happens to an unfinished goal on resume is undocumented | Lifecycle | — | Low–medium. |

## Status legend

- `open`: observed and unresolved.
- `workaround`: we have a client-side workaround that costs us something.
- `blocking`: no client-side fix is possible; it needs a Muse Code change.

Every issue file carries its status in its header.
