# [T-1] Workspace skeleton

Milestone: M0 · Type: chore · Layer: all · Size: small · Status: todo

## Goal

Cargo workspace with all seven crates, so later tasks only add code.

## Scope

- Crates `domain`, `application`, `infra-audio`, `infra-demucs`, `infra-storage`, `cli`, `web` under `crates/`.
- `cli` builds the `noiseapp` binary, `web` builds `noiseapp-web`; the rest are libraries.
- Path dependencies exactly as in the CLAUDE.md table.
- `.gitignore`, `rustfmt.toml`, workspace clippy lints, `rust-toolchain.toml` pinning a stable release.

## Out of scope

- CI (T-2), dependency guard (T-3), domain code, `frontend/` (M5), `tools/export/` (M1).

## Acceptance criteria

- [ ] `cargo build --workspace` passes on Windows and Linux and produces `noiseapp` and `noiseapp-web`.
- [ ] `cargo fmt --check` and `cargo clippy --workspace --all-targets -- -D warnings` pass.
- [ ] Crate dependencies match the CLAUDE.md table.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
