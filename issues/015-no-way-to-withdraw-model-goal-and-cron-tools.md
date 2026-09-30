# 015: Withdrawing the model's goal and schedule tools needs a full allowlist (no denylist, no per-session policy)

- **Status:** open, medium. CORRECTED 2026-10-01: a home-level allowlist works; asks narrowed
- **Area:** tools / goals / host control
- **Observed on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-10-01

## Correction (2026-10-01)

Our first report said there was no supported way. That was wrong. The
settings key `run.toolset` in `settings.json` takes a named allowlist of
tools. Listing every wanted tool except `create_goal`, `cron_create` and
`cron_delete` removes those three from the session toolset (25 tools down to
18 in our test). An unknown tool name exits with an error, so the setting
fails closed. What remains:
- **No denylist.** An allowlist must name every other tool. It drops tools the
  host did not think of (in our test `read_skill`, `work_status`,
  `snooze_reminder` and `write_todos`), and it goes stale as Muse Code adds
  tools.
- **No per-session policy over MSP.** `session/start` config carries only
  `mcpServers`, so the policy is home-wide.
- **Not yet checked:** whether a workflow or a subagent can still reach goal
  creation when the parent's toolset excludes it.

## Original report

The model in a Muse Code session is offered `create_goal`, `update_goal`,
`get_goal` and `report_progress`, plus `cron_create`, `cron_delete` and
`cron_list`. An embedding host that owns the session goal (it sets the goal
over MSP, enforces its own budget and reads it back) has no supported way to
withdraw the goal-creating and schedule-creating tools:
- MSP `session/start` config carries only `mcpServers`, with no tool policy.
- A top-level `disabled_tools` member in `settings.json` (a string that exists
  in the binary) is rejected at startup as an unknown settings member. The
  session's host toolset snapshot still lists `create_goal`, `cron_create` and
  `cron_delete`.

## Impact on an embedding client

- The model can create a goal the host did not set. That goal carries no host
  budget, and native continuation keeps working on it.
- The model can schedule its own future wake-ups.

Both bypass the host's goal plane. The only safe response a host has is
reactive: detect the foreign goal, clear it and interrupt the turn. Or it can
stop using native goals altogether and deliver its objective as plain text,
which gives up the durable goal identity, the structured statuses and the
read-after-write acknowledgement.

## What would help

- A denylist form (for example `run.disabled_tools`) next to
  `run.toolset`.
- A per-session tool policy in `session/start`.
- Documentation of `run.toolset`, including how it applies to subagents and
  workflows.
- Alternatively, a host capability that makes goals host-owned only: model
  tools may update or report progress but not create or replace a goal, and
  may not schedule.
