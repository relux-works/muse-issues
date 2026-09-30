# 006: The `clientInfo.name` pattern is enforced at runtime but missing from the schema

- **Status:** workaround
- **Area:** MSP schema / `initialize`
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1)

## Summary

`initialize` with `clientInfo.name` containing a hyphen (for example
`my-client`) is rejected by runtime validation that requires
`^[a-z0-9_]+$`. The exported JSON schema does not express this pattern, so a
schema-validating client passes its own checks and still fails against the
server. The process exited 1 in our smoke.

## Impact

Low. It only cost debugging time, but it is the kind of contract drift that
typed clients generated from the schema cannot catch.

## What would help

Add the `pattern` to `clientInfo.name` in the exported schema, and return a
typed, descriptive error instead of exiting.
