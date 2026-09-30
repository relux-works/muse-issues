# 006: The `clientInfo.name` pattern is enforced at runtime but missing from the schema (WITHDRAWN)

- **Status:** withdrawn. This was our error.
- **Area:** MSP schema / `initialize`

## Correction (2026-09-30)

Our original report said that the exported schema does not express the
`clientInfo.name` pattern. That is wrong. `ClientInfo.name` carries
`pattern: ^[a-z0-9_]+$` in the schemas exported from 1.3.0, 1.4.0 and 1.4.1.
On 1.4.1 an `initialize` with `probe-client` returns a typed, descriptive error:

```text
-32602 invalid initialize params: clientInfo.name must be a machine identifier
matching ^[a-z0-9_]+$ (SS1.4.1)   data.kind = invalidParams
```

Nothing to fix on the Muse Code side. We keep this file so the numbering stays
stable.
