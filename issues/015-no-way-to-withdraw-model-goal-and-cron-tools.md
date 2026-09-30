# 015: No supported way to withdraw the model's goal and schedule tools from a session

- **Status:** open, high for any embedding host that owns the goal
- **Area:** tools / goals / host control
- **Observed on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-10-01

## Summary

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

- A per-session tool policy in `session/start` (for example,
  `disabledTools`), or a documented settings key, that removes named tools
  from the model's toolset.
- Alternatively, a host capability that makes goals host-owned only: model
  tools may update or report progress but not create or replace a goal, and
  may not schedule.
