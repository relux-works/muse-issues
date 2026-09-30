# 013: `serve` has no `--no-foreign-personal-context` (only `exec` does)

- **Status:** open, medium–high for isolated headless hosts
- **Area:** serve flags / isolation
- **Observed on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-09-30

## Summary

`muse exec --help` lists `--no-foreign-personal-context` ("Exclude foreign
personal rules and skills from this run"). `muse serve --help` does not.

## Impact

A headless host that runs `serve` for automated work cannot exclude the
operator's personal rules and skills from other tools. Runs are then less
reproducible, and the model sees instructions it was not given. The only
workaround is a fully private `HOME`/XDG tree, which in turn loses the model
catalog and credentials (see [004](004-echo-provider-not-fully-offline.md) and
[005](005-macos-keychain-blocks-headless-serve.md)).

## What would help

Flag parity: accept `--no-foreign-personal-context` on `serve`, or as a
per-session option in `session/start`.
