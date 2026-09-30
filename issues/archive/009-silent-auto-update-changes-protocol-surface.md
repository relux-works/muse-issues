# 009: The launcher auto-updates silently under a running integration

- **Status:** low (pinning works; the compatibility signal already exists)
- **Area:** distribution / reproducibility
- **Observed on:** the installed launcher updated itself from 1.3.0 to 1.4.0
  between two of our work sessions
- **Re-verified on:** 2026-09-30. It happened again: the launcher moved from 1.4.0 to 1.4.1 (1.4.1-R4503.1) on its own.

## Summary

Our protocol research was done against 1.3.0. Later, the installed `muse`
launcher had auto-updated to 1.4.0 without any action on our side. For an
integration that speaks a versioned protocol, a silent update of the server
under a running integration is a reproducibility problem. In this case the
client-facing MSP shapes turned out unchanged between 1.3.0 and 1.4.0, but we
only learned that after exporting and diffing both schemas.

## Workaround we use

We run a pinned copy of the versioned binary (never the auto-updating launcher
path), always with `MUSE_NO_AUTO_UPDATE=1`, and we record the server version
from `initialize` (`serverInfo.version`).

## What would help

- A documented, supported way to pin a version for embedded/headless use.
- ~~A protocol-level compatibility signal in `initialize`~~: this already
  exists as `InitializeResult.schema {version, fingerprint}`, and the
  fingerprint changed from 1.4.0 (`99a7458c…`) to 1.4.1 (`e0e163db…`). We had
  missed it. Remaining ask: publish the fingerprint per release, and document
  `MUSE_NO_AUTO_UPDATE=1` plus the versioned `muse-bin-<version>` binary as
  the supported pin.
