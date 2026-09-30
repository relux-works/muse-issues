# 012: Subagent and workflow token usage is excluded from `session/tokenUsage.cumulative`

- **Status:** open, high for any budget-enforcing client
- **Area:** usage accounting
- **Observed on:** the Muse Code 1.4.1 schema (documented behaviour)

## Summary

The schema says `session/tokenUsage.cumulative` never folds in
subagent/workflow-child usage: "Subagent/workflow-child usage is never folded
in — it rides the owning items" (`SessionTokenUsageParams.cumulative`, and
`Item.usage` for subagent items).

## Impact

A controller that enforces a token budget per goal from
`session/tokenUsage` does not see tokens spent by subagents or workflows
spawned inside that goal. The budget is silently exceeded.

## What would help

A transitive, session-wide cumulative (parent plus children) alongside the
parent-only one, or per-child `tokenUsage` events that a client can sum.
