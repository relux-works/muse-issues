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
  otherwise. One supervised real-model run (2026-09-30) used the default
  `meta` provider with `muse-spark-1.3-contributor` on 1.4.1.
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
  added 010–014. 006 was withdrawn: it was our error. On 2026-09-30 we moved
  002, 006, 007, 009 and 010 to the archive.

## About Muse Code issue references

We have no access to the Muse Code team's tracker. Every `#NNNNN` or `tdd SS…`
reference in these files is quoted from the descriptions in the MSP schema
exported from the 1.4.0 binary, so it can be matched to internal work items.

## Active issues

| # | Title | Area | Muse Code refs | Impact on us |
| --- | --- | --- | --- | --- |
| [015](issues/015-no-way-to-withdraw-model-goal-and-cron-tools.md) | Withdrawing the model's `create_goal` and `cron_*` tools needs a full allowlist (`run.toolset`) | Tools / goals | — | Medium. Works; we ask for a denylist and a per-session policy. |
| [012](issues/012-subagent-usage-not-in-session-cumulative.md) | Subagent/workflow usage is excluded from the session cumulative | Usage | — | **High** for budget enforcement: child usage is invisible to a per-goal budget. |
| [005](issues/005-macos-keychain-blocks-headless-serve.md) | On macOS an auth file in an isolated `HOME` makes `muse serve` hang on a Keychain UI prompt | Headless / macOS | — | **High** for isolated headless hosts on macOS. A normal logged-in `HOME` works. |
| [004](issues/004-echo-provider-not-fully-offline.md) | The offline `echo` provider refuses turns (`authRequired`) and emits no usage | Testing | — | Medium–high. The usage/budget path cannot be tested offline. |
| [013](issues/013-serve-lacks-no-foreign-personal-context.md) | `serve` lacks `--no-foreign-personal-context` (only `exec` has it) | Isolation | — | Medium–high. It forces a fully private `HOME`, which runs into 005. |
| [008](issues/008-userinputdialogs-default-capable.md) | Must-answer server requests reach headless clients (`approval/request` has no opt-out) | MSP capabilities | #23840 · SS5.3 | Medium. Worked around by refusing server requests; the default `allowAll` avoids most of them. |
| [001](issues/001-goal-ack-not-correlatable.md) | A goal command's effect is not correlated to the command on the wire | MSP goals | #33066 · SS3.18, SS4.6.2 | Medium. Worked around (read-after-write plus replay by `commandId`); we want `goalId`/`revision` on the wire. |
| [011](issues/011-identical-goal-adoption-has-no-noop-ack.md) | Identical goal adoption emits nothing and has no `noop` ack | MSP goals | SS4.6.2 | Medium. Worked around with a read. |
| [003](issues/003-goal-state-machine-undocumented.md) | The goal state machine (`blocked`, `-32030`) is undocumented and asynchronous | MSP goals | #33066 · SS3.18, SS4.6.2 | Low–medium. A documentation ask; worked around. |
| [014](issues/014-resume-does-not-rewake-unfinished-goal.md) | What happens to an unfinished goal on resume is undocumented | Lifecycle | — | Low–medium. A documentation ask. |
| [016](issues/016-retained-frame-digest-input-undocumented.md) | The `content_sha256` of a retained session-log frame has no documented input | Session log | — | Low. Worked around: spelling checked, content unverified. |

None of the active issues blocks us outright today: each has a workaround or
only limits isolation, testing or budget accuracy.

## Archive

Kept for the record and for stable numbering. They are no longer active.

| # | Title | Why archived |
| --- | --- | --- |
| [002](issues/archive/002-session-read-has-no-goal.md) | `session/read` has no goal | Corrected: `session/resume` with `history:"snapshot"` returns the goal. Minor residual ask: a lease-free goal read. |
| [006](issues/archive/006-clientinfo-name-regex-not-in-schema.md) | The `clientInfo.name` pattern is missing from the schema | Withdrawn: our error; the schema has the pattern. |
| [007](issues/archive/007-no-session-log-suppresses-goalchanged.md) | `serve --no-session-log` suppresses `goalChanged` | By design (ephemeral profile, announced in `initialize`). Minor docs ask. |
| [009](issues/archive/009-silent-auto-update-changes-protocol-surface.md) | The launcher auto-updates silently | Pinning works, and `initialize` already reports `schema.fingerprint`. |
| [010](issues/archive/010-goal-set-response-waits-for-woken-turn.md) | The `goal/set` response waits for the woken turn | Not reproduced with a real provider (response in about 0.6 s); an `echo` artifact. |

## Status legend

- `open`: observed and unresolved.
- `workaround`: we have a client-side workaround that costs us something.
- `blocking`: no client-side fix is possible; it needs a Muse Code change.

Every issue file carries its status in its header.
