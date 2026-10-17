# 017: interactive `-c context.foreign_personal_skills=false` still loads a foreign skill

- **Status:** open, observation note (not yet reported to the vendor)
- **Area:** interactive TUI / isolation
- **Observed on:** Muse Code 1.4.8 (1.4.8-R6256.1), binary SHA-256
  `5721763b4723f961709cd28f69aa049763965b46c4d4d70f883a0a90e5ddf39c`,
  macOS arm64, 2026-10-16

## Summary

This is a note on behaviour we observe, kept so we can recheck it on the next
build. On 1.4.8 the exec form excludes foreign personal skills, but the
interactive form does not, even with the override that is meant to do it:

| Form | Command shape | Native `AGENTS.md` present | Foreign skills loaded |
| --- | --- | --- | --- |
| exec | `muse exec --json --provider echo --trust-workspace --no-foreign-personal-context ...` | yes | 0 (expected) |
| interactive | `muse --provider echo --trust-workspace -c context.foreign_personal_skills=false` | yes | **1** (expected 0) |

The interactive result is the same whether the native root is a symlink or a
regular file. The TUI accepts the override (exit 0, no parser error), shows
the foreign skill in its startup banner and in `/rules`, and the
`skills.catalog_load` record still counts it.

Without a native `AGENTS.md`, the rules fallback is active and the foreign
skill also stays in the interactive form. The same was true on 1.4.7.

## How we observed it

- `MUSE_NO_AUTO_UPDATE=1`, `env -i`, a fresh `HOME` and fresh XDG directories,
  `TBH_CREDENTIAL_BACKEND=file`, `--provider echo`.
- An OS sandbox denies network and Keychain access, so no login or remote
  inference takes part.
- One synthetic foreign skill in an isolated foreign source, plus a native
  `AGENTS.md` under the Muse XDG config directory.
- The TUI runs in a private background terminal. Input: a local echo turn,
  then `/rules`, then `/exit`.
- Controls: without the override the same skill loads in both forms, so a zero
  count cannot pass by accident. Both the exec-only flag and the string value
  `"false"` are refused by the TUI parser with exit 2, before the loader runs.

## Relation to 1.4.7

In an earlier, less controlled 1.4.7 probe (banner only, on a real home), the
same `-c context.foreign_personal_skills=false` with a native `AGENTS.md`
present removed the skills line from the banner. We did not replay that probe
on 1.4.7 under the controlled setup above, so we cannot say for sure that 1.4.8
is a regression.

## Impact

An interactive session started with this override still sees the operator's
personal skills from other tools. For an isolated, reproducible interactive
session, we treat 1.4.8's interactive form as not excluding foreign skills.
`exec` is unaffected. Related: [013](013-serve-lacks-no-foreign-personal-context.md),
where `serve` has no exclusion flag at all.

## What would help

The interactive override `context.foreign_personal_skills=false` should
exclude foreign personal skills, as `--no-foreign-personal-context` does for
`exec`. If the override is not meant to work this way, a note in the
documentation would help.
