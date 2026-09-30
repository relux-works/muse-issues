# 014: What happens to an unfinished goal when a session is resumed is undocumented

- **Status:** open, low–medium (a documentation ask; the behaviour is only partly observed)
- **Area:** session lifecycle
- **Observed on:** Muse Code 1.4.1 (1.4.1-R4503.1), `echo` provider, 2026-09-30

## Summary

Host A held a session with a goal and was then killed (`SIGKILL`). Host B then
called `session/resume` on the same session:
- it succeeded (while A was alive, the same call returned `-32021 sessionInUse`);
- the snapshot carried the goal intact;
- no goal turn was woken, and the only event was `session/branchChanged`.

**Caveat:** on `echo` every turn fails, so the goal was already `blocked` when
the host was killed. We therefore do not know whether a goal that is still
`active` at crash time is re-woken on resume.

## Impact

A controller that restores a session after a crash does not know whether to
re-arm the goal itself (for example with `goal/resume` or `goal/set`). Guessing
wrong either leaves the goal idle or, combined with
[011](011-identical-goal-adoption-has-no-noop-ack.md), turns a re-apply of the
same goal into an apparently lost ack.

## What would help

Document the resume behaviour per goal status (`active`, `blocked`, `paused`):
whether a goal turn is woken, and whether the client is expected to re-arm it.
