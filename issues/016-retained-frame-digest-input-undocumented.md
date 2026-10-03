# 016: The `content_sha256` of a retained session-log frame has no documented input

- **Status:** open, low. A documentation ask; worked around.
- **Area:** session log / integrity
- **Observed on:** Muse Code 1.4.1 (1.4.1-R4503.1), 2026-10-02. Not yet
  re-checked on 1.4.2 (1.4.2-R4684.1, now on `muse-stable`).

## Observed behaviour

The interactive TUI writes a per-session event log,
`YYYY/MM/DD/<session>/session.jsonl`. Most lines are single records. A
permission change is written as a retained frame instead:

```json
{"retained_frame": "session_permission_transaction",
 "frame_schema_version": 1,
 "transaction_id": "...",
 "outer_log_ordinal": 1,
 "children": [
   {"child_index": 0, "record_json": "{...permission_format_declared...}"},
   {"child_index": 1, "record_json": "{...permission_profile_committed...}"}],
 "content_sha256": "sha256:<64 hex>"}
```

`content_sha256` looks like an integrity digest over the frame's content, but
nothing we have says which bytes it covers. We tried 16 candidate inputs
(15 distinct) on each of the 3 frames captured from real sessions. None
reproduced any captured digest:
- the `record_json` strings concatenated: raw, newline-joined, and
  newline-terminated;
- the parsed children as canonical JSON (sorted keys, no whitespace): each
  child concatenated, and as an array with and without spaces;
- the `children` array as canonical JSON, and as `[{"record_json": ...}]`;
- the `record_json` strings as a JSON string array, with and without spaces;
- `transaction_id` before and after the concatenated `record_json` strings;
- the SHA-256 hex of each `record_json`, concatenated;
- the whole frame as canonical JSON without `content_sha256`, and the whole
  frame re-serialized as compact JSON in its original key order.

## Impact on an embedding client

A client that reads the session log to decide when the session is idle (so
that it can safely type into the TUI) cannot verify the frame. It has to
choose between two options:
- **Refusing frames it cannot verify.** A session that ever showed a
  permission dialog never reads as idle again.
- **Accepting the digest unverified.** This is what we do. We check only its
  spelling (`sha256:` plus 64 hex digits). We validate the children
  independently and never treat the digest as proof of content integrity.

## Workaround

As above: accept the frame with the digest unverified, spelling checked.

## Request

Document the digest input: exactly which bytes are hashed, in which
serialization and in which order. Then a reader of the log can verify a frame
fail-closed. If the digest is internal and not meant to be verified by
readers, saying so would also settle it.
