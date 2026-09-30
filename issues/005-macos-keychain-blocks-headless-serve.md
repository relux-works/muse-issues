# 005: On macOS an auth file makes `muse serve` hang on a Keychain UI prompt

- **Status:** workaround (we avoid credentials entirely)
- **Area:** headless operation / macOS auth storage
- **Observed on:** Muse Code 1.4.0 (1.4.0-R4302.1), macOS arm64

## Summary

In an isolated `HOME`, with no provider configured, `muse serve` answers
`initialize` normally. Adding only an `auth.json` fixture (a fixed dummy
offline credential, shaped like the SDK quickstart fixture) makes the server
**stop responding**:
- `initialize` is never answered;
- stderr is empty.

A process sample shows the main thread inside the macOS Security framework,
waiting on an interactive authorization UI:

```text
SecItemAdd
 SecItemAdd_osx
  SecKeychainItemCreateFromContent
   Security::KeychainCore::StorageManager::defaultKeychainUI
    Security::KeychainCore::StorageManager::makeLoginAuthUI
```

Pointing the process at an unlocked temporary keychain did not avoid the UI
path.

## Impact on an embedding client

A headless or CI process cannot run `muse serve` with any credential on macOS:
it blocks forever on a dialog nobody can see, without an error. We had to drop
credentials from all automated runs, which is also why we depend on `echo`
([004](004-echo-provider-not-fully-offline.md)).

## What would help

- A supported headless auth store (file-based, or an env/flag that disables the
  Keychain import) for non-interactive processes.
- Fail fast with a clear error when an interactive Keychain prompt would be
  needed and there is no UI session (for example when serving over stdio).
